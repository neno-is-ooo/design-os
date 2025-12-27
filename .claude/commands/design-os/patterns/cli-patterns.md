# CLI Design Patterns

Reference patterns for command-line interface applications.

## Command Structure Patterns

### Single Command
- One primary action
- Flags modify behavior
- Example: `cat`, `ls`, `curl`

```
mytool [flags] [arguments]
mytool --verbose input.txt
```

### Subcommand Pattern
- Multiple related commands
- Each subcommand has own flags
- Example: `git`, `docker`, `npm`

```
mytool <command> [flags] [arguments]
mytool init --template basic
mytool build --release
mytool deploy --env production
```

### Nested Subcommands
- Commands have subcommands
- Example: `kubectl get pods`, `aws s3 ls`

```
mytool <group> <command> [flags]
mytool config get key
mytool config set key value
```

## Argument Patterns

### Positional Arguments
- Order matters
- Required arguments first
- Optional arguments last

```
mytool <required> [optional]
mytool input.txt output.txt
```

### Flags (Boolean)
- Short form: `-v`
- Long form: `--verbose`
- Combined short: `-vvv` (stacking)

```
mytool -v                # short
mytool --verbose         # long
mytool -abc              # combined (-a -b -c)
```

### Options (With Values)
- Short: `-o value` or `-ovalue`
- Long: `--output value` or `--output=value`

```
mytool -o file.txt
mytool --output file.txt
mytool --output=file.txt
```

### Standard Flags
Always support these:
- `-h, --help`: Show help
- `-v, --version`: Show version
- `-q, --quiet`: Suppress output
- `--verbose`: Increase output
- `--dry-run`: Preview without executing
- `--config FILE`: Custom config file

## Output Patterns

### Human-Readable (Default)
```
Processing 3 files...
  ✓ file1.txt (2.3 KB)
  ✓ file2.txt (1.1 KB)
  ✗ file3.txt: Permission denied

Completed: 2 succeeded, 1 failed
```

### Machine-Readable (--json)
```json
{
  "processed": 3,
  "succeeded": 2,
  "failed": 1,
  "results": [
    {"file": "file1.txt", "status": "success", "size": 2355},
    {"file": "file2.txt", "status": "success", "size": 1124},
    {"file": "file3.txt", "status": "error", "error": "Permission denied"}
  ]
}
```

### Quiet Mode (-q)
- Only output essential data
- No progress, no decoration
- Good for piping to other tools

### Verbose Mode (-v, -vv, -vvv)
- `-v`: Basic info
- `-vv`: Detailed info
- `-vvv`: Debug info

## Progress Indication

### Spinner
```
⠋ Processing...
⠙ Processing...
⠹ Processing...
```

### Progress Bar
```
Processing: [████████████░░░░░░░░] 60% (3/5)
```

### Status Updates
```
[1/5] Downloading dependencies...
[2/5] Building project...
[3/5] Running tests...
```

## Error Handling

### Exit Codes
- `0`: Success
- `1`: General error
- `2`: Misuse (bad arguments)
- `126`: Permission denied
- `127`: Command not found

### Error Messages
```
Error: Could not read 'config.json': File not found

Try: mytool --help for usage information
```

### Suggestions
```
Error: Unknown command 'biuld'

Did you mean?
  build
  bundle
```

## Interactive Patterns

### Confirmation Prompts
```
This will delete 23 files. Continue? [y/N]
```

### Selection Menu
```
Select environment:
  ❯ development
    staging
    production
```

### Input Prompts
```
Project name: my-project
Description: A cool project
Author [John Doe]:
```

### Password Input
```
Password: ••••••••
```

## Configuration Patterns

### Config File Locations
1. Command flag: `--config path`
2. Environment variable: `MYTOOL_CONFIG`
3. Local: `./.mytoolrc` or `./mytool.config.json`
4. User: `~/.config/mytool/config.json`
5. System: `/etc/mytool/config.json`

### Environment Variables
```
MYTOOL_TOKEN=xxx      # Authentication
MYTOOL_DEBUG=1        # Enable debug
MYTOOL_NO_COLOR=1     # Disable colors
```

### Dotfiles
```
# .mytoolrc
verbose = true
output = json

[defaults]
timeout = 30
```

## Help Text Pattern

```
mytool - A tool for doing things

USAGE:
  mytool <command> [flags]

COMMANDS:
  init        Create a new project
  build       Build the project
  deploy      Deploy to environment
  help        Show help for a command

FLAGS:
  -h, --help       Show this help message
  -v, --version    Show version
  -q, --quiet      Suppress output
  --verbose        Increase output verbosity
  --config FILE    Use custom config file

EXAMPLES:
  mytool init my-project
  mytool build --release
  mytool deploy --env production

For more help on a command, run:
  mytool help <command>
```

## Color Usage

### Semantic Colors
- **Green**: Success, created, added
- **Red**: Error, deleted, failed
- **Yellow**: Warning, modified
- **Blue**: Info, links
- **Cyan**: Paths, commands
- **Gray**: Secondary info

### Respect NO_COLOR
```bash
# Check before using colors
if [ -z "$NO_COLOR" ] && [ -t 1 ]; then
  # Use colors
fi
```

## Piping & Composition

### Stdin Support
```bash
cat file.txt | mytool process
echo "data" | mytool format
```

### Stdout for Piping
- Data to stdout
- Messages to stderr
- Exit code for status

```bash
mytool list | grep pattern | wc -l
```

### Here Documents
```bash
mytool create <<EOF
name: my-project
type: library
EOF
```

## Autocomplete

### Bash Completion
```bash
_mytool_completion() {
  local commands="init build deploy"
  COMPREPLY=($(compgen -W "$commands" -- "${COMP_WORDS[COMP_CWORD]}"))
}
complete -F _mytool_completion mytool
```

### Generate Completions
```bash
mytool completion bash > /etc/bash_completion.d/mytool
mytool completion zsh > ~/.zsh/completions/_mytool
mytool completion fish > ~/.config/fish/completions/mytool.fish
```

## Testing CLI Applications

### Unit Tests
- Test individual functions
- Mock file system, network
- Test argument parsing

### Integration Tests
- Run actual commands
- Check exit codes
- Verify output

### Snapshot Tests
- Capture help text
- Compare output format
- Detect unintended changes

## Packaging Considerations

### Binary Distribution
- Single binary (Go, Rust)
- No runtime dependencies
- Cross-compile for platforms

### Script Distribution
- Shebang line
- Check dependencies
- Install script

### Package Managers
- Homebrew (macOS)
- apt/yum (Linux)
- npm/pip (language-specific)
