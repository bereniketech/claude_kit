---
name: perl-patterns
description: Comprehensive Perl skill covering modern idioms, strict/warnings, references, regex, Moo OO, CPAN modules, taint mode, security (input validation, eval avoidance), and Test2/Test::More.
---

# Perl Development Patterns

Modern Perl 5.36+ for clean, secure, testable applications — from language idioms to security hardening and TDD.

---

## 1. Modern Preamble and Signatures

Use `use v5.36` instead of legacy boilerplate. It enables `strict`, `warnings`, and subroutine signatures automatically.

```perl
# Good: single pragma
use v5.36;

sub greet($name) { say "Hello, $name!" }

sub connect_db($host, $port = 5432, $timeout = 30) {
    return DBI->connect("dbi:Pg:host=$host;port=$port", undef, undef, {
        RaiseError => 1, PrintError => 0,
    });
}

# Bad: legacy boilerplate
use strict; use warnings;
use feature 'say', 'signatures';
no warnings 'experimental::signatures';
```

Use postfix dereferencing (`$ref->@*`, `$ref->%*`) over circumfix. Use `isa` operator (5.32+) instead of `blessed($o) && $o->isa('X')`.

**Rule:** Never use `no strict 'refs'`. Never use indirect object syntax (`new Foo`). Use `Foo->new`.

---

## 2. References and Data Structures

```perl
use v5.36;

my $config = {
    database => { host => 'localhost', port => 5432 },
};

# Safe deep access — returns undef if any level missing
my $port = $config->{database}{port};

# Postfix dereference
my @users  = $data->{users}->@*;
my %first  = $data->{users}[0]->%*;
my @roles  = $data->{users}[0]{roles}->@*;

# Hash slices
my @vals = @{$config->{database}}{qw(host port)};
```

---

## 3. Regular Expressions

Use named captures with `/x` for readability. Precompile patterns used in loops. Use possessive quantifiers or atomic groups to prevent ReDoS.

```perl
use v5.36;

my $log_re = qr{
    ^ (?<timestamp> \d{4}-\d{2}-\d{2} \s \d{2}:\d{2}:\d{2} )
    \s+ \[ (?<level> \w+ ) \]
    \s+ (?<message> .+ ) $
}x;

if ($line =~ $log_re) {
    say "Level: $+{level}, Message: $+{message}";
}

# Precompile once
my $email_re = qr/^[A-Za-z0-9._%+-]+\@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/;

# Possessive — prevents catastrophic backtracking
my $safe_re = qr/^[a-zA-Z]++$/;
```

**Rule:** Never use nested quantifiers on overlapping patterns (`(a+)+`). Always anchor patterns from user input.

---

## 4. Modern OO with Moo

Prefer `Moo` for lightweight OO with typed attributes. Use `Moo::Role` for composable behavior. Use `namespace::autoclean` to prevent method pollution.

```perl
package User;
use Moo;
use Types::Standard qw(Str Int ArrayRef);
use namespace::autoclean;

has name  => (is => 'ro', isa => Str, required => 1);
has email => (is => 'ro', isa => Str, required => 1);
has age   => (is => 'ro', isa => Int, default  => sub { 0 });
has roles => (is => 'ro', isa => ArrayRef[Str], default => sub { [] });

sub is_admin($self) { grep { $_ eq 'admin' } $self->roles->@* }

1;
```

Use `Moo::Role` with `requires` for interface contracts. Use Moose only when the full metaprotocol is needed.

---

## 5. Error Handling

Use `Try::Tiny` for reliable exception handling (5.36). Use native `try/catch` on 5.40+. Always `eval` file I/O and external calls.

```perl
use v5.36;
use Try::Tiny;

sub fetch_user($id) {
    my $user = try {
        $db->resultset('User')->find($id)
            // die "User $id not found\n";
    }
    catch { warn "Failed to fetch user $id: $_"; undef };
    return $user;
}
```

**Rule:** Never use string `eval` with user input — that is remote code execution. Use `Module::Runtime::require_module` for dynamic module loading.

---

## 6. Taint Mode and Input Security

Enable taint mode (`-T`) for all web-facing and CGI scripts. Sanitize `PATH` and delete dangerous env vars immediately.

```perl
#!/usr/bin/perl -T
use v5.36;

$ENV{PATH} = '/usr/local/bin:/usr/bin:/bin';
delete @ENV{qw(IFS CDPATH ENV BASH_ENV)};

# Untaint with specific, restrictive regex — $1 is untainted
sub untaint_username($input) {
    $input =~ /^([a-zA-Z0-9_]{3,30})$/ or die "Invalid username\n";
    return $1;
}

# Bad: overly permissive — defeats the purpose
sub bad_untaint($input) { $input =~ /^(.*)$/s; return $1 }
```

**Rule:** Allowlists over blocklists. Validate sort fields, column names, and enum values against a `%allowed` hash before use in queries.

---

## 7. Safe File and Process Operations

```perl
use v5.36;
use autodie;
use File::Spec;
use Cwd qw(realpath);

# Three-arg open — mandatory
open my $fh, '<:encoding(UTF-8)', $path;

# Path traversal guard
sub safe_path($base, $user_path) {
    my $real      = realpath(File::Spec->catfile($base, $user_path)) // die "Invalid path\n";
    my $base_real = realpath($base) // die "Invalid base\n";
    die "Path traversal blocked\n" unless $real =~ /^\Q$base_real\E(?:\/|\z)/;
    return $real;
}

# List-form system — no shell interpolation
system('grep', '-r', $pattern, '/var/log/app/') == 0
    or die "Command failed\n";

# Never: string-form system
# system("grep -r '$pattern' /var/log/");  # shell injection!
```

**Rule:** Never use two-arg open with user data. Never use backticks with interpolated variables. Use `IPC::Run3` or `Capture::Tiny` to capture subprocess output safely.

---

## 8. SQL Injection Prevention

Always use DBI placeholders. Validate column/direction names against an allowlist before interpolating into SQL.

```perl
use v5.36;
use DBI;

# Good: parameterized
sub find_user($dbh, $email) {
    my $sth = $dbh->prepare('SELECT * FROM users WHERE email = ?');
    $sth->execute($email);
    return $sth->fetchrow_hashref;
}

# Good: allowlisted column for dynamic ORDER BY
sub order_by($dbh, $col, $dir) {
    my %ok_cols = map { $_ => 1 } qw(name email created_at);
    my %ok_dirs = map { $_ => 1 } qw(ASC DESC);
    die "Invalid\n" unless $ok_cols{$col} && $ok_dirs{uc $dir};
    $dbh->prepare("SELECT * FROM users ORDER BY $col $dir")->execute;
}

# Bad: interpolation — SQLi
# $dbh->do("DELETE FROM users WHERE id = $id");
```

Use `DBIx::Class` for ORM-level safety: it generates parameterized queries automatically.

---

## 9. Testing with Test2::V0 and TDD

Follow RED-GREEN-REFACTOR. Use `Test2::V0` for new code; `Test::More` only for legacy. Use `prove -l` to include `lib/` in `@INC`.

```perl
use v5.36;
use Test2::V0;
use lib 'lib';
use MyApp::User;

subtest 'User creation' => sub {
    my $user = User->new(name => 'Alice', email => 'alice@example.com');
    is($user->name, 'Alice', 'name set');
    ok($user isa 'User', 'correct class');
};

# Deep comparison with builders
is(
    $result->to_hash,
    hash {
        field name  => 'Alice';
        field email => match(qr/\@example\.com$/);
        etc();
    },
    'result matches expected shape'
);

# Exception testing
like(dies { divide(10, 0) }, qr/Division by zero/, 'dies on zero');
ok(lives { divide(10, 2) }, 'succeeds on valid input');

done_testing;
```

Use `Test::MockModule` for mocking — never monkey-patch without restoration. Use in-memory SQLite for database integration tests.

```bash
prove -lr t/          # all tests
prove -lv t/unit/     # verbose unit tests
prove -lr -j8 t/      # parallel
cover -test && cover -report html   # coverage
```

**Rule:** Always end test files with `done_testing`. Always use `my` (not `our`) inside subtests to prevent state leakage. Target 80%+ coverage.
