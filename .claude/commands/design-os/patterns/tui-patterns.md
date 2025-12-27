# TUI (Terminal UI) Design Patterns

Comprehensive patterns for the Charm ecosystem and other TUI frameworks.

---

## Charm Ecosystem Overview

The Charm ecosystem provides a complete toolkit for building terminal applications in Go.

### Core Libraries

| Library | Purpose | Use When |
|---------|---------|----------|
| **[Bubble Tea](https://github.com/charmbracelet/bubbletea)** | TUI framework (Elm architecture) | Building interactive terminal apps |
| **[Lip Gloss](https://github.com/charmbracelet/lipgloss)** | Declarative styling | Styling any terminal output |
| **[Bubbles](https://github.com/charmbracelet/bubbles)** | Pre-built components | Lists, tables, inputs, spinners, etc. |
| **[Huh](https://github.com/charmbracelet/huh)** | Forms and prompts | User input, wizards, accessible forms |

### Content Rendering

| Library | Purpose | Use When |
|---------|---------|----------|
| **[Glamour](https://github.com/charmbracelet/glamour)** | Markdown rendering | Displaying formatted markdown |
| **[Glow](https://github.com/charmbracelet/glow)** | Markdown reader CLI | Reading/browsing markdown files |

### Infrastructure

| Library | Purpose | Use When |
|---------|---------|----------|
| **[Wish](https://github.com/charmbracelet/wish)** | SSH server | Serving TUIs over SSH |
| **[Wishlist](https://github.com/charmbracelet/wishlist)** | SSH directory | Multi-app SSH endpoint |
| **[Log](https://github.com/charmbracelet/log)** | Colorful logging | Structured, styled logs |

### Shell Scripting

| Library | Purpose | Use When |
|---------|---------|----------|
| **[Gum](https://github.com/charmbracelet/gum)** | Shell script TUI | Adding TUI to bash/shell scripts |

### Tooling

| Library | Purpose | Use When |
|---------|---------|----------|
| **[VHS](https://github.com/charmbracelet/vhs)** | Terminal recording | Creating demo GIFs/videos |
| **[Freeze](https://github.com/charmbracelet/freeze)** | Terminal screenshots | Code/output images |

### AI Integration

| Library | Purpose | Use When |
|---------|---------|----------|
| **[Mods](https://github.com/charmbracelet/mods)** | LLM in pipelines | AI-powered CLI workflows |
| **[Crush](https://github.com/charmbracelet/crush)** | AI coding agent | Terminal-based AI assistant |

### Experimental (x/)

| Package | Purpose |
|---------|---------|
| **[x/teatest](https://pkg.go.dev/github.com/charmbracelet/x/teatest)** | Testing Bubble Tea programs |
| **[x/term](https://pkg.go.dev/github.com/charmbracelet/x/term)** | Terminal utilities |
| **[x/input](https://pkg.go.dev/github.com/charmbracelet/x/input)** | Input handling |
| **[x/ansi](https://pkg.go.dev/github.com/charmbracelet/x/ansi)** | ANSI escape sequences |

---

## Other TUI Frameworks

| Framework | Language | Architecture | Best For |
|-----------|----------|--------------|----------|
| **[Textual](https://github.com/Textualize/textual)** | Python | Widget-based, CSS-like | Python apps, rapid prototyping |
| **[Ratatui](https://github.com/ratatui-org/ratatui)** | Rust | Immediate mode | Rust apps, performance |
| **[Ink](https://github.com/vadimdemedes/ink)** | JS/TS | React-like | JS/TS apps, React developers |

---

## Bubble Tea Core Patterns

### The Elm Architecture

Bubble Tea uses Model → Update → View:

```go
package main

import (
    "fmt"
    tea "github.com/charmbracelet/bubbletea"
)

// Model holds application state
type model struct {
    items    []string
    cursor   int
    selected map[int]struct{}
}

// Init returns initial command (can be nil)
func (m model) Init() tea.Cmd {
    return nil
}

// Update handles messages and returns updated model + command
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "q", "ctrl+c":
            return m, tea.Quit
        case "up", "k":
            if m.cursor > 0 {
                m.cursor--
            }
        case "down", "j":
            if m.cursor < len(m.items)-1 {
                m.cursor++
            }
        case "enter", " ":
            if _, ok := m.selected[m.cursor]; ok {
                delete(m.selected, m.cursor)
            } else {
                m.selected[m.cursor] = struct{}{}
            }
        }
    }
    return m, nil
}

// View renders UI as a string
func (m model) View() string {
    s := "Select items:\n\n"
    for i, item := range m.items {
        cursor := " "
        if m.cursor == i {
            cursor = ">"
        }
        checked := " "
        if _, ok := m.selected[i]; ok {
            checked = "x"
        }
        s += fmt.Sprintf("%s [%s] %s\n", cursor, checked, item)
    }
    s += "\nPress q to quit.\n"
    return s
}

func main() {
    p := tea.NewProgram(model{
        items:    []string{"Item 1", "Item 2", "Item 3"},
        selected: make(map[int]struct{}),
    })
    if _, err := p.Run(); err != nil {
        fmt.Println("Error:", err)
    }
}
```

### Full-Window vs Inline Mode

```go
// Full-window (default) - takes over terminal
p := tea.NewProgram(model{})

// Inline - renders in place, doesn't clear screen
p := tea.NewProgram(model{}, tea.WithoutClearScreen())

// Alt screen buffer - preserves terminal content on exit
p := tea.NewProgram(model{}, tea.WithAltScreen())

// Mouse support
p := tea.NewProgram(model{}, tea.WithMouseCellMotion())
```

### Sub-Model Composition

```go
// Parent model containing child models
type model struct {
    list     list.Model      // Bubbles list component
    input    textinput.Model // Bubbles text input
    viewport viewport.Model  // Bubbles viewport
    state    appState
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    var cmds []tea.Cmd

    switch m.state {
    case stateList:
        var cmd tea.Cmd
        m.list, cmd = m.list.Update(msg)
        cmds = append(cmds, cmd)
    case stateInput:
        var cmd tea.Cmd
        m.input, cmd = m.input.Update(msg)
        cmds = append(cmds, cmd)
    }

    return m, tea.Batch(cmds...)
}
```

---

## Huh: Forms and Prompts

Huh is a dedicated forms library with accessibility support and theming.

### Basic Form

```go
import "github.com/charmbracelet/huh"

var name string
var email string

form := huh.NewForm(
    huh.NewGroup(
        huh.NewInput().
            Title("Name").
            Value(&name),
        huh.NewInput().
            Title("Email").
            Value(&email).
            Validate(func(s string) error {
                if !strings.Contains(s, "@") {
                    return errors.New("invalid email")
                }
                return nil
            }),
    ),
)

err := form.Run()
```

### Field Types

```go
// Text input
huh.NewInput().
    Title("Username").
    Placeholder("Enter username...").
    Suggestions([]string{"admin", "user", "guest"}).
    Value(&username)

// Text area (multi-line)
huh.NewText().
    Title("Description").
    Lines(5).
    Value(&description)

// Select (single choice)
huh.NewSelect[string]().
    Title("Color").
    Options(
        huh.NewOption("Red", "red"),
        huh.NewOption("Green", "green"),
        huh.NewOption("Blue", "blue"),
    ).
    Value(&color)

// Multi-select
huh.NewMultiSelect[string]().
    Title("Toppings").
    Options(
        huh.NewOption("Cheese", "cheese").Selected(true),
        huh.NewOption("Pepperoni", "pepperoni"),
        huh.NewOption("Mushrooms", "mushrooms"),
    ).
    Value(&toppings)

// Confirm
huh.NewConfirm().
    Title("Are you sure?").
    Affirmative("Yes").
    Negative("No").
    Value(&confirmed)
```

### Multi-Page Forms (Groups)

```go
form := huh.NewForm(
    // Page 1: Account info
    huh.NewGroup(
        huh.NewInput().Title("Username").Value(&username),
        huh.NewInput().Title("Email").Value(&email),
    ).Title("Account"),

    // Page 2: Preferences
    huh.NewGroup(
        huh.NewSelect[string]().Title("Theme").Options(...).Value(&theme),
        huh.NewConfirm().Title("Notifications").Value(&notifications),
    ).Title("Preferences"),
)
```

### Theming

```go
// Built-in themes
form.WithTheme(huh.ThemeDracula())
form.WithTheme(huh.ThemeCatppuccin())
form.WithTheme(huh.ThemeBase16())
form.WithTheme(huh.ThemeCharm()) // default

// Custom theme
theme := huh.ThemeBase()
theme.Focused.Title = lipgloss.NewStyle().Foreground(lipgloss.Color("205"))
form.WithTheme(theme)
```

### Accessibility Mode

```go
// Enable accessible mode for screen readers
form.WithAccessible(true)

// Or via environment variable
// ACCESSIBLE=true ./myapp
```

### Huh Spinner

```go
import "github.com/charmbracelet/huh/spinner"

action := func() {
    time.Sleep(2 * time.Second)
}

spinner.New().
    Title("Processing...").
    Action(action).
    Run()
```

### Embedding in Bubble Tea

```go
type model struct {
    form *huh.Form
}

func (m model) Init() tea.Cmd {
    return m.form.Init()
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    form, cmd := m.form.Update(msg)
    if f, ok := form.(*huh.Form); ok {
        m.form = f
    }

    if m.form.State == huh.StateCompleted {
        // Form submitted
    }

    return m, cmd
}

func (m model) View() string {
    return m.form.View()
}
```

---

## Bubbles Components

### List

```go
import "github.com/charmbracelet/bubbles/list"

// Define item type
type item struct {
    title, desc string
}

func (i item) Title() string       { return i.title }
func (i item) Description() string { return i.desc }
func (i item) FilterValue() string { return i.title }

// Create list
items := []list.Item{
    item{title: "Item 1", desc: "Description 1"},
    item{title: "Item 2", desc: "Description 2"},
}

l := list.New(items, list.NewDefaultDelegate(), 30, 20)
l.Title = "My List"
l.SetShowStatusBar(true)
l.SetFilteringEnabled(true)
```

### Table

```go
import "github.com/charmbracelet/bubbles/table"

columns := []table.Column{
    {Title: "Name", Width: 20},
    {Title: "Status", Width: 10},
    {Title: "Updated", Width: 15},
}

rows := []table.Row{
    {"Project Alpha", "Active", "2 hours ago"},
    {"Project Beta", "Paused", "1 day ago"},
}

t := table.New(
    table.WithColumns(columns),
    table.WithRows(rows),
    table.WithFocused(true),
    table.WithHeight(10),
)

// Styling
s := table.DefaultStyles()
s.Header = s.Header.Bold(true).BorderBottom(true)
s.Selected = s.Selected.Background(lipgloss.Color("205"))
t.SetStyles(s)
```

### Text Input

```go
import "github.com/charmbracelet/bubbles/textinput"

ti := textinput.New()
ti.Placeholder = "Enter text..."
ti.Focus()
ti.CharLimit = 156
ti.Width = 40

// Password input
ti.EchoMode = textinput.EchoPassword
ti.EchoCharacter = '•'

// Suggestions
ti.ShowSuggestions = true
ti.SetSuggestions([]string{"apple", "banana", "cherry"})
```

### Text Area

```go
import "github.com/charmbracelet/bubbles/textarea"

ta := textarea.New()
ta.Placeholder = "Enter description..."
ta.SetWidth(60)
ta.SetHeight(10)
ta.Focus()

// Character limit
ta.CharLimit = 500
ta.ShowLineNumbers = true
```

### Viewport (Scrollable Content)

```go
import "github.com/charmbracelet/bubbles/viewport"

vp := viewport.New(80, 20)
vp.SetContent(longContent)

// In Update
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.WindowSizeMsg:
        m.viewport.Width = msg.Width
        m.viewport.Height = msg.Height - 4 // Leave room for header/footer
    }

    var cmd tea.Cmd
    m.viewport, cmd = m.viewport.Update(msg)
    return m, cmd
}
```

### Progress Bar

```go
import "github.com/charmbracelet/bubbles/progress"

// Gradient progress bar
p := progress.New(progress.WithDefaultGradient())

// Solid color
p := progress.New(progress.WithSolidFill("#FF6B9D"))

// Custom colors
p := progress.New(progress.WithGradient("#FF6B9D", "#9D4EDD"))

// Update progress (0.0 to 1.0)
p.SetPercent(0.5)

// Animated
cmd := p.SetPercent(0.75) // Returns animation command
```

### Spinner

```go
import "github.com/charmbracelet/bubbles/spinner"

s := spinner.New()
s.Spinner = spinner.Dot
s.Style = lipgloss.NewStyle().Foreground(lipgloss.Color("205"))

// Available spinners
spinner.Line
spinner.Dot
spinner.MiniDot
spinner.Jump
spinner.Pulse
spinner.Points
spinner.Globe
spinner.Moon
spinner.Monkey
spinner.Meter
spinner.Hamburger
```

### File Picker

```go
import "github.com/charmbracelet/bubbles/filepicker"

fp := filepicker.New()
fp.CurrentDirectory, _ = os.UserHomeDir()
fp.AllowedTypes = []string{".go", ".mod", ".sum"}
fp.ShowHidden = false
```

### Timer / Stopwatch

```go
import "github.com/charmbracelet/bubbles/timer"
import "github.com/charmbracelet/bubbles/stopwatch"

// Countdown timer
t := timer.NewWithInterval(5*time.Minute, time.Second)

// Stopwatch
sw := stopwatch.NewWithInterval(time.Second)
```

### Help

```go
import "github.com/charmbracelet/bubbles/help"
import "github.com/charmbracelet/bubbles/key"

type keyMap struct {
    Up    key.Binding
    Down  key.Binding
    Quit  key.Binding
}

func (k keyMap) ShortHelp() []key.Binding {
    return []key.Binding{k.Up, k.Down, k.Quit}
}

func (k keyMap) FullHelp() [][]key.Binding {
    return [][]key.Binding{
        {k.Up, k.Down},
        {k.Quit},
    }
}

keys := keyMap{
    Up: key.NewBinding(
        key.WithKeys("up", "k"),
        key.WithHelp("↑/k", "up"),
    ),
    Down: key.NewBinding(
        key.WithKeys("down", "j"),
        key.WithHelp("↓/j", "down"),
    ),
    Quit: key.NewBinding(
        key.WithKeys("q", "ctrl+c"),
        key.WithHelp("q", "quit"),
    ),
}

h := help.New()
h.View(keys) // Renders help text
```

---

## Lip Gloss Styling

### Basic Styles

```go
import "github.com/charmbracelet/lipgloss"

style := lipgloss.NewStyle().
    Bold(true).
    Foreground(lipgloss.Color("205")).
    Background(lipgloss.Color("236")).
    Padding(1, 2).
    Margin(1).
    Width(40)

output := style.Render("Hello, World!")
```

### Colors

```go
// ANSI 16 colors (most compatible)
lipgloss.Color("1")  // Red
lipgloss.Color("2")  // Green

// ANSI 256 colors
lipgloss.Color("99")  // Purple

// True color (hex)
lipgloss.Color("#FF6B9D")

// Adaptive (light/dark terminal)
lipgloss.AdaptiveColor{
    Light: "#1A1A1A",
    Dark:  "#FAFAFA",
}

// Complete adaptive (16/256/true color)
lipgloss.CompleteAdaptiveColor{
    Light: lipgloss.CompleteColor{
        TrueColor: "#1A1A1A",
        ANSI256:   "234",
        ANSI:      "0",
    },
    Dark: lipgloss.CompleteColor{
        TrueColor: "#FAFAFA",
        ANSI256:   "255",
        ANSI:      "15",
    },
}
```

### Borders

```go
// Border types
lipgloss.NormalBorder()
lipgloss.RoundedBorder()
lipgloss.ThickBorder()
lipgloss.DoubleBorder()
lipgloss.HiddenBorder()

style := lipgloss.NewStyle().
    Border(lipgloss.RoundedBorder()).
    BorderForeground(lipgloss.Color("62")).
    BorderBackground(lipgloss.Color("236"))

// Individual sides
style.BorderTop(true).BorderBottom(true)
```

### Layout

```go
// Vertical join
lipgloss.JoinVertical(
    lipgloss.Left,   // Alignment
    header,
    content,
    footer,
)

// Horizontal join
lipgloss.JoinHorizontal(
    lipgloss.Top,    // Alignment
    sidebar,
    main,
)

// Place (position within space)
lipgloss.Place(
    width, height,
    lipgloss.Center, lipgloss.Center,
    content,
)

// Place with whitespace options
lipgloss.Place(
    width, height,
    lipgloss.Center, lipgloss.Center,
    dialog,
    lipgloss.WithWhitespaceChars("░"),
    lipgloss.WithWhitespaceForeground(subtle),
)
```

### Tables (Lip Gloss v2)

```go
import "github.com/charmbracelet/lipgloss/table"

t := table.New().
    Border(lipgloss.NormalBorder()).
    BorderStyle(lipgloss.NewStyle().Foreground(lipgloss.Color("99"))).
    Headers("Name", "Age", "City").
    Row("Alice", "30", "NYC").
    Row("Bob", "25", "LA")

fmt.Println(t)
```

---

## Glamour: Markdown Rendering

```go
import "github.com/charmbracelet/glamour"

// Simple render
out, err := glamour.Render(markdown, "dark")

// With options
r, _ := glamour.NewTermRenderer(
    glamour.WithAutoStyle(),        // Auto-detect light/dark
    glamour.WithWordWrap(80),
    glamour.WithEmoji(),
)
out, err := r.Render(markdown)

// Available styles
glamour.DarkStyle
glamour.LightStyle
glamour.PinkStyle
glamour.DraculaStyle
glamour.TokyoNightStyle
glamour.NoTTYStyle
```

---

## Wish: SSH Server

Serve TUIs over SSH.

```go
import (
    "github.com/charmbracelet/wish"
    "github.com/charmbracelet/wish/bubbletea"
)

func main() {
    s, err := wish.NewServer(
        wish.WithAddress(":2222"),
        wish.WithHostKeyPath(".ssh/term_info_ed25519"),
        wish.WithMiddleware(
            bubbletea.Middleware(teaHandler),
            logging.Middleware(),
        ),
    )

    log.Printf("Starting SSH server on :2222")
    log.Fatal(s.ListenAndServe())
}

func teaHandler(s ssh.Session) (tea.Model, []tea.ProgramOption) {
    return model{}, []tea.ProgramOption{tea.WithAltScreen()}
}
```

### Wish Middleware

```go
// Git server
wish.WithMiddleware(git.Middleware(repoDir, nil))

// Access control
wish.WithMiddleware(accesscontrol.Middleware())

// Active terminal required
wish.WithMiddleware(activeterm.Middleware())

// Elapsed time logging
wish.WithMiddleware(elapsed.Middleware())
```

---

## Gum: Shell Script TUI

Add TUI to shell scripts without Go.

### Input

```bash
# Simple input
NAME=$(gum input --placeholder "Enter name")

# With suggestions
COLOR=$(gum input --placeholder "Color" --suggestions "red,green,blue")

# Password
PASSWORD=$(gum input --password --placeholder "Password")
```

### Choose

```bash
# Single select
COLOR=$(gum choose "red" "green" "blue")

# Multi-select
TOPPINGS=$(gum choose --no-limit "cheese" "pepperoni" "mushrooms")

# With limit
ITEMS=$(gum choose --limit 3 "a" "b" "c" "d" "e")
```

### Confirm

```bash
gum confirm "Are you sure?" && rm -rf ./data
```

### Filter

```bash
# Fuzzy filter from input
ls | gum filter

# With placeholder
SELECTED=$(cat files.txt | gum filter --placeholder "Select file...")
```

### Spin

```bash
gum spin --spinner dot --title "Processing..." -- sleep 5
```

### Write (Multi-line)

```bash
DESCRIPTION=$(gum write --placeholder "Enter description...")
```

### Format

```bash
gum format "# Heading" "Some **bold** text"
gum format --type code "func main() { }"
```

### Style

```bash
gum style --foreground 212 --bold "Hello"
gum style --border double --padding "1 2" "Boxed content"
```

### File

```bash
FILE=$(gum file .)
FILE=$(gum file --directory --all /path)
```

### Pager

```bash
cat large-file.txt | gum pager
```

### Log

```bash
gum log --level info "Starting process"
gum log --level error "Something went wrong"
```

### Table

```bash
gum table < data.csv
echo "Name,Age\nAlice,30\nBob,25" | gum table
```

---

## Log: Structured Logging

```go
import "github.com/charmbracelet/log"

// Basic usage
log.Info("Starting server", "port", 8080)
log.Warn("Connection slow", "latency", "200ms")
log.Error("Failed to connect", "err", err)

// Levels
log.SetLevel(log.DebugLevel)

// Custom logger
logger := log.NewWithOptions(os.Stderr, log.Options{
    ReportTimestamp: true,
    TimeFormat:      time.Kitchen,
    Prefix:          "myapp",
})

// With context
logger.With("request_id", "abc123").Info("Handling request")

// Formatters
log.SetFormatter(log.JSONFormatter)
log.SetFormatter(log.LogfmtFormatter)
log.SetFormatter(log.TextFormatter) // default
```

---

## Testing with teatest

```go
import "github.com/charmbracelet/x/teatest"

func TestApp(t *testing.T) {
    m := initialModel()
    tm := teatest.NewTestModel(t, m, teatest.WithInitialTermSize(80, 24))

    // Send key
    tm.Send(tea.KeyMsg{Type: tea.KeyDown})

    // Wait for condition
    teatest.WaitFor(t, tm.Output(), func(bts []byte) bool {
        return strings.Contains(string(bts), "expected text")
    })

    // Send quit
    tm.Send(tea.KeyMsg{Type: tea.KeyRunes, Runes: []rune("q")})

    // Verify final output
    tm.WaitFinished(t, teatest.WithFinalTimeout(time.Second))

    out := tm.FinalOutput(t)
    if !strings.Contains(string(out), "goodbye") {
        t.Error("expected goodbye message")
    }
}
```

### Golden File Testing

```go
func TestView(t *testing.T) {
    m := model{items: []string{"one", "two"}, cursor: 0}

    got := m.View()

    // First run creates golden file, subsequent runs compare
    teatest.RequireEqualOutput(t, []byte(got))
}
```

---

## Mouse Support

### Enable Mouse

```go
p := tea.NewProgram(model{}, tea.WithMouseCellMotion())
// or
p := tea.NewProgram(model{}, tea.WithMouseAllMotion())
```

### Handle Mouse Events

```go
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.MouseMsg:
        switch msg.Button {
        case tea.MouseButtonLeft:
            if msg.Action == tea.MouseActionPress {
                m.handleClick(msg.X, msg.Y)
            }
        case tea.MouseButtonWheelUp:
            m.scroll(-1)
        case tea.MouseButtonWheelDown:
            m.scroll(1)
        }
    }
    return m, nil
}
```

### BubbleZone (Click Zones)

```go
import zone "github.com/lrstanley/bubblezone"

// In View
func (m model) View() string {
    // Wrap clickable areas
    button := zone.Mark("my-button", buttonStyle.Render("Click me"))
    return button
}

// In Update
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.MouseMsg:
        if zone.Get("my-button").InBounds(msg) {
            // Button was clicked
        }
    }
    return m, nil
}

// Wrap program
p := tea.NewProgram(model{}, tea.WithMouseCellMotion())
zone.NewGlobal()
```

---

## Async Operations

### Commands

```go
// Fetch data
func fetchData() tea.Msg {
    resp, err := http.Get("https://api.example.com/data")
    if err != nil {
        return errMsg{err}
    }
    defer resp.Body.Close()

    var data Data
    json.NewDecoder(resp.Body).Decode(&data)
    return dataMsg{data}
}

// Use in Init or Update
func (m model) Init() tea.Cmd {
    return fetchData
}

// Handle result
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case dataMsg:
        m.data = msg.data
        m.loading = false
    case errMsg:
        m.err = msg.err
    }
    return m, nil
}
```

### Batch Commands

```go
func (m model) Init() tea.Cmd {
    return tea.Batch(
        fetchUsers,
        fetchProjects,
        m.spinner.Tick,
    )
}
```

### Tick / Timer

```go
func tickCmd() tea.Cmd {
    return tea.Tick(time.Second, func(t time.Time) tea.Msg {
        return tickMsg(t)
    })
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tickMsg:
        m.elapsed++
        return m, tickCmd() // Keep ticking
    }
    return m, nil
}
```

---

## Layout Patterns

### Full-Screen with Header/Footer

```go
func (m model) View() string {
    return lipgloss.JoinVertical(
        lipgloss.Left,
        m.headerView(),
        m.contentView(),
        m.footerView(),
    )
}

func (m model) headerView() string {
    title := titleStyle.Render("My Application")
    return headerStyle.Width(m.width).Render(title)
}

func (m model) footerView() string {
    help := "↑/↓: navigate • enter: select • q: quit"
    return footerStyle.Width(m.width).Render(help)
}
```

### Split Panes

```go
func (m model) View() string {
    sidebar := sidebarStyle.
        Width(30).
        Height(m.height).
        Render(m.sidebarView())

    content := contentStyle.
        Width(m.width - 30).
        Height(m.height).
        Render(m.contentView())

    return lipgloss.JoinHorizontal(lipgloss.Top, sidebar, content)
}
```

### Tabs

```go
func (m model) View() string {
    var tabs []string
    for i, t := range m.tabs {
        if i == m.activeTab {
            tabs = append(tabs, activeTabStyle.Render(t))
        } else {
            tabs = append(tabs, tabStyle.Render(t))
        }
    }

    tabBar := lipgloss.JoinHorizontal(lipgloss.Top, tabs...)
    content := m.tabContent()

    return lipgloss.JoinVertical(lipgloss.Left, tabBar, content)
}
```

### Modal Dialog

```go
func (m model) View() string {
    main := m.mainView()

    if m.showDialog {
        dialog := dialogStyle.Render(m.dialogContent())

        // Center dialog over main
        return lipgloss.Place(
            m.width, m.height,
            lipgloss.Center, lipgloss.Center,
            dialog,
            lipgloss.WithWhitespaceChars("░"),
            lipgloss.WithWhitespaceForeground(lipgloss.Color("238")),
        )
    }

    return main
}
```

---

## Window Size Handling

```go
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.WindowSizeMsg:
        m.width = msg.Width
        m.height = msg.Height

        // Resize components
        m.list.SetSize(msg.Width, msg.Height-4)
        m.viewport.Width = msg.Width
        m.viewport.Height = msg.Height - 6
        m.help.Width = msg.Width
    }
    return m, nil
}
```

---

## Color Themes

### ANSI Palette

```go
var (
    // Standard 16 colors
    black   = lipgloss.Color("0")
    red     = lipgloss.Color("1")
    green   = lipgloss.Color("2")
    yellow  = lipgloss.Color("3")
    blue    = lipgloss.Color("4")
    magenta = lipgloss.Color("5")
    cyan    = lipgloss.Color("6")
    white   = lipgloss.Color("7")

    // Bright variants (8-15)
    brightBlack   = lipgloss.Color("8")
    brightRed     = lipgloss.Color("9")
    brightGreen   = lipgloss.Color("10")
    brightYellow  = lipgloss.Color("11")
    brightBlue    = lipgloss.Color("12")
    brightMagenta = lipgloss.Color("13")
    brightCyan    = lipgloss.Color("14")
    brightWhite   = lipgloss.Color("15")
)
```

### Theme Structure

```go
type Theme struct {
    Primary    lipgloss.TerminalColor
    Secondary  lipgloss.TerminalColor
    Background lipgloss.TerminalColor
    Foreground lipgloss.TerminalColor
    Muted      lipgloss.TerminalColor
    Success    lipgloss.TerminalColor
    Warning    lipgloss.TerminalColor
    Error      lipgloss.TerminalColor
    Border     lipgloss.TerminalColor
}

var DarkTheme = Theme{
    Primary:    lipgloss.Color("205"),
    Secondary:  lipgloss.Color("39"),
    Background: lipgloss.Color("236"),
    Foreground: lipgloss.Color("252"),
    Muted:      lipgloss.Color("241"),
    Success:    lipgloss.Color("82"),
    Warning:    lipgloss.Color("214"),
    Error:      lipgloss.Color("196"),
    Border:     lipgloss.Color("238"),
}

var LightTheme = Theme{
    Primary:    lipgloss.Color("205"),
    Secondary:  lipgloss.Color("33"),
    Background: lipgloss.Color("255"),
    Foreground: lipgloss.Color("234"),
    Muted:      lipgloss.Color("245"),
    Success:    lipgloss.Color("34"),
    Warning:    lipgloss.Color("172"),
    Error:      lipgloss.Color("160"),
    Border:     lipgloss.Color("250"),
}
```

---

## VHS: Terminal Recording

Create demo GIFs with `.tape` files:

```tape
# demo.tape
Output demo.gif

Set FontSize 14
Set Width 1200
Set Height 600
Set Theme "Catppuccin Mocha"

Type "my-cli --help"
Enter
Sleep 2s

Type "my-cli create project"
Enter
Sleep 1s

Type "my-project"
Enter
Sleep 3s
```

Run: `vhs demo.tape`

---

## Freeze: Terminal Screenshots

```bash
# Screenshot command output
freeze --execute "my-cli --help" -o help.png

# Screenshot code file
freeze main.go -o code.png

# Interactive mode
freeze --interactive

# With options
freeze main.go \
    --theme "catppuccin-mocha" \
    --window \
    --shadow.blur 20 \
    --font.family "JetBrains Mono" \
    -o screenshot.png
```

---

## Accessibility

### Screen Reader Support (Huh)

```go
form := huh.NewForm(groups...).WithAccessible(true)
```

### High Contrast

```go
// Use adaptive colors
text := lipgloss.AdaptiveColor{Light: "#000000", Dark: "#FFFFFF"}

// Ensure sufficient contrast
style := lipgloss.NewStyle().
    Foreground(lipgloss.Color("15")).  // White
    Background(lipgloss.Color("0"))    // Black
```

### Keyboard Navigation

- All actions must be keyboard-accessible
- Provide visible focus indicators
- Support standard navigation (arrows, tab, enter, esc)
- Document keybindings with help component

---

## Performance

- **Minimize re-renders**: Only update what changed
- **Use viewports**: Don't render off-screen content
- **Cache styles**: Create styles once, reuse
- **Batch commands**: Use `tea.Batch()` for multiple concurrent operations
- **Profile**: Use `pprof` for bottlenecks
- **Memoize**: TextArea component now includes memoization (v2)

---

## Other Frameworks

### Textual (Python)

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Static, Button
from textual.containers import Container

class MyApp(App):
    CSS = """
    Screen {
        layout: vertical;
    }
    #main {
        height: 1fr;
    }
    """

    BINDINGS = [("q", "quit", "Quit")]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Container(
            Static("Hello, World!"),
            Button("Click me", id="btn"),
            id="main"
        )
        yield Footer()

    def on_button_pressed(self, event: Button.Pressed) -> None:
        self.notify("Button clicked!")
```

### Ratatui (Rust)

```rust
use ratatui::{
    prelude::*,
    widgets::{Block, Borders, Paragraph},
};

fn ui(frame: &mut Frame, app: &App) {
    let chunks = Layout::default()
        .direction(Direction::Vertical)
        .constraints([
            Constraint::Length(3),
            Constraint::Min(1),
            Constraint::Length(3),
        ])
        .split(frame.area());

    let header = Block::default()
        .borders(Borders::ALL)
        .title("Header");
    frame.render_widget(header, chunks[0]);

    let content = Paragraph::new(app.content.clone())
        .block(Block::default().borders(Borders::ALL));
    frame.render_widget(content, chunks[1]);
}
```

### Ink (React for Terminal)

```tsx
import React, { useState } from 'react';
import { render, Box, Text, useInput } from 'ink';

const App = () => {
    const [selected, setSelected] = useState(0);
    const items = ['Item 1', 'Item 2', 'Item 3'];

    useInput((input, key) => {
        if (key.upArrow) setSelected(s => Math.max(0, s - 1));
        if (key.downArrow) setSelected(s => Math.min(items.length - 1, s + 1));
    });

    return (
        <Box flexDirection="column">
            {items.map((item, i) => (
                <Text key={i} color={i === selected ? 'green' : undefined}>
                    {i === selected ? '> ' : '  '}{item}
                </Text>
            ))}
        </Box>
    );
};

render(<App />);
```

---

## Resources

- [Charm Homepage](https://charm.sh)
- [Bubble Tea Docs](https://pkg.go.dev/github.com/charmbracelet/bubbletea)
- [Bubble Tea Examples](https://github.com/charmbracelet/bubbletea/tree/master/examples)
- [Bubbles Components](https://github.com/charmbracelet/bubbles)
- [Lip Gloss Docs](https://pkg.go.dev/github.com/charmbracelet/lipgloss)
- [Huh Forms](https://github.com/charmbracelet/huh)
- [Awesome Bubble Tea](https://github.com/charmbracelet/bubbletea#bubble-tea-in-the-wild)
