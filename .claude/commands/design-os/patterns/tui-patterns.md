# TUI (Terminal UI) Design Patterns

Reference patterns for interactive terminal user interface applications. Focuses on the Charm ecosystem (Bubble Tea, Lip Gloss, Bubbles) with notes for other frameworks.

## Framework Overview

| Framework | Language | Architecture | Best For |
|-----------|----------|--------------|----------|
| **Bubble Tea** | Go | Elm-like (Model-Update-View) | Full TUI apps |
| **Lip Gloss** | Go | Declarative styling | Styling for Bubble Tea |
| **Bubbles** | Go | Pre-built components | Common UI patterns |
| **Textual** | Python | Widget-based, CSS-like | Python apps, rapid prototyping |
| **Ratatui** | Rust | Immediate mode | Rust apps, performance |
| **Ink** | JS/TS | React-like | JS/TS apps, React developers |

---

## Elm Architecture (Bubble Tea)

Bubble Tea uses The Elm Architecture: Model → Update → View

```go
// Model holds application state
type model struct {
    items    []string
    cursor   int
    selected map[int]struct{}
}

// Init returns initial model and optional command
func (m model) Init() tea.Cmd {
    return nil
}

// Update handles messages and returns updated model
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

// View renders the UI as a string
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
```

---

## Layout Patterns

### Full-Screen Application

```go
// Main layout with header, content, footer
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
    return headerStyle.Render(title)
}

func (m model) footerView() string {
    help := "↑/↓: navigate • enter: select • q: quit"
    return footerStyle.Render(help)
}
```

### Split Panes (Sidebar + Content)

```go
func (m model) View() string {
    sidebar := m.sidebarView()
    content := m.contentView()

    return lipgloss.JoinHorizontal(
        lipgloss.Top,
        sidebarStyle.Width(30).Render(sidebar),
        contentStyle.Render(content),
    )
}
```

### Tabs

```go
type model struct {
    tabs     []string
    activeTab int
}

func (m model) View() string {
    var renderedTabs []string
    for i, tab := range m.tabs {
        if i == m.activeTab {
            renderedTabs = append(renderedTabs, activeTabStyle.Render(tab))
        } else {
            renderedTabs = append(renderedTabs, tabStyle.Render(tab))
        }
    }
    tabBar := lipgloss.JoinHorizontal(lipgloss.Top, renderedTabs...)
    content := m.tabContent()

    return lipgloss.JoinVertical(lipgloss.Left, tabBar, content)
}
```

---

## Component Patterns

### List with Selection

```go
import "github.com/charmbracelet/bubbles/list"

type model struct {
    list list.Model
}

func (m model) Init() tea.Cmd {
    return nil
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    var cmd tea.Cmd
    m.list, cmd = m.list.Update(msg)
    return m, cmd
}

func (m model) View() string {
    return m.list.View()
}
```

### Text Input

```go
import "github.com/charmbracelet/bubbles/textinput"

type model struct {
    input textinput.Model
}

func initialModel() model {
    ti := textinput.New()
    ti.Placeholder = "Enter text..."
    ti.Focus()
    ti.CharLimit = 156
    ti.Width = 40
    return model{input: ti}
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    var cmd tea.Cmd
    m.input, cmd = m.input.Update(msg)
    return m, cmd
}
```

### Form with Multiple Inputs

```go
type model struct {
    inputs  []textinput.Model
    focused int
}

func (m *model) nextInput() {
    m.focused = (m.focused + 1) % len(m.inputs)
}

func (m *model) prevInput() {
    m.focused--
    if m.focused < 0 {
        m.focused = len(m.inputs) - 1
    }
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "tab", "down":
            m.nextInput()
        case "shift+tab", "up":
            m.prevInput()
        }
    }

    // Update focused input
    cmd := m.updateInputs(msg)
    return m, cmd
}
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
```

### Viewport (Scrollable Content)

```go
import "github.com/charmbracelet/bubbles/viewport"

type model struct {
    viewport viewport.Model
    content  string
}

func (m model) Init() tea.Cmd {
    return nil
}

func (m *model) setSize(width, height int) {
    m.viewport.Width = width
    m.viewport.Height = height - 4 // Leave room for header/footer
    m.viewport.SetContent(m.content)
}

func (m model) View() string {
    return fmt.Sprintf("%s\n%s\n%s",
        m.headerView(),
        m.viewport.View(),
        m.footerView(),
    )
}
```

### Progress Bar / Spinner

```go
import (
    "github.com/charmbracelet/bubbles/progress"
    "github.com/charmbracelet/bubbles/spinner"
)

// Spinner for indeterminate loading
s := spinner.New()
s.Spinner = spinner.Dot
s.Style = lipgloss.NewStyle().Foreground(lipgloss.Color("205"))

// Progress bar for determinate progress
p := progress.New(progress.WithDefaultGradient())
p.SetPercent(0.5)
```

---

## Styling with Lip Gloss

### Basic Styles

```go
import "github.com/charmbracelet/lipgloss"

// Colors
var (
    primary   = lipgloss.Color("205")  // Pink
    secondary = lipgloss.Color("240")  // Gray
    success   = lipgloss.Color("82")   // Green
    warning   = lipgloss.Color("214")  // Orange
    danger    = lipgloss.Color("196")  // Red
)

// Styles
var (
    titleStyle = lipgloss.NewStyle().
        Bold(true).
        Foreground(primary).
        MarginBottom(1)

    selectedStyle = lipgloss.NewStyle().
        Foreground(lipgloss.Color("229")).
        Background(primary).
        Padding(0, 1)

    normalStyle = lipgloss.NewStyle().
        Foreground(secondary)

    helpStyle = lipgloss.NewStyle().
        Foreground(lipgloss.Color("241"))
)
```

### Borders and Boxes

```go
var boxStyle = lipgloss.NewStyle().
    Border(lipgloss.RoundedBorder()).
    BorderForeground(lipgloss.Color("62")).
    Padding(1, 2).
    Width(40)

var dialogStyle = lipgloss.NewStyle().
    Border(lipgloss.DoubleBorder()).
    BorderForeground(lipgloss.Color("205")).
    Padding(1, 2).
    Align(lipgloss.Center)
```

### Adaptive Colors (Light/Dark Terminal)

```go
var (
    // AdaptiveColor picks based on terminal background
    subtle = lipgloss.AdaptiveColor{Light: "#D9DCCF", Dark: "#383838"}
    text   = lipgloss.AdaptiveColor{Light: "#1A1A1A", Dark: "#FAFAFA"}
)

var adaptiveStyle = lipgloss.NewStyle().
    Foreground(text).
    Background(subtle)
```

---

## Navigation Patterns

### Focus Management

```go
type focusable int

const (
    focusSidebar focusable = iota
    focusContent
    focusDialog
)

type model struct {
    focus focusable
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.KeyMsg:
        switch msg.String() {
        case "tab":
            m.focus = (m.focus + 1) % 3
        case "shift+tab":
            m.focus--
            if m.focus < 0 {
                m.focus = 2
            }
        }
    }

    // Route input to focused component
    switch m.focus {
    case focusSidebar:
        return m.updateSidebar(msg)
    case focusContent:
        return m.updateContent(msg)
    case focusDialog:
        return m.updateDialog(msg)
    }
    return m, nil
}
```

### Modal Dialogs

```go
type model struct {
    showDialog bool
    dialogMsg  string
}

func (m model) View() string {
    main := m.mainView()

    if m.showDialog {
        dialog := m.dialogView()
        // Center dialog over main content
        return lipgloss.Place(
            m.width, m.height,
            lipgloss.Center, lipgloss.Center,
            dialog,
            lipgloss.WithWhitespaceChars(" "),
            lipgloss.WithWhitespaceForeground(subtle),
        )
    }
    return main
}
```

---

## Keyboard Handling

### Standard Key Bindings

```go
// Common conventions
case "q", "ctrl+c":
    return m, tea.Quit
case "?":
    m.showHelp = !m.showHelp
case "esc":
    m.showDialog = false
case "enter":
    // Confirm/select
case "up", "k":
    // Move up (vim-style k)
case "down", "j":
    // Move down (vim-style j)
case "left", "h":
    // Move left / collapse
case "right", "l":
    // Move right / expand
case "g":
    // Go to top
case "G":
    // Go to bottom
case "/":
    // Search
case "n":
    // Next search result
case "N":
    // Previous search result
```

### Key Bindings Help

```go
import "github.com/charmbracelet/bubbles/key"

type keyMap struct {
    Up    key.Binding
    Down  key.Binding
    Help  key.Binding
    Quit  key.Binding
}

func (k keyMap) ShortHelp() []key.Binding {
    return []key.Binding{k.Help, k.Quit}
}

func (k keyMap) FullHelp() [][]key.Binding {
    return [][]key.Binding{
        {k.Up, k.Down},
        {k.Help, k.Quit},
    }
}

var keys = keyMap{
    Up: key.NewBinding(
        key.WithKeys("up", "k"),
        key.WithHelp("↑/k", "move up"),
    ),
    Down: key.NewBinding(
        key.WithKeys("down", "j"),
        key.WithHelp("↓/j", "move down"),
    ),
    Help: key.NewBinding(
        key.WithKeys("?"),
        key.WithHelp("?", "toggle help"),
    ),
    Quit: key.NewBinding(
        key.WithKeys("q", "ctrl+c"),
        key.WithHelp("q", "quit"),
    ),
}
```

---

## Async Operations

### Commands (Side Effects)

```go
// Commands return messages
func fetchData() tea.Msg {
    resp, err := http.Get("https://api.example.com/data")
    if err != nil {
        return errMsg{err}
    }
    // Parse and return data message
    return dataMsg{data}
}

// Tick for periodic updates
func tickCmd() tea.Cmd {
    return tea.Tick(time.Second, func(t time.Time) tea.Msg {
        return tickMsg(t)
    })
}

func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case dataMsg:
        m.data = msg.data
        m.loading = false
    case errMsg:
        m.err = msg.err
        m.loading = false
    case tickMsg:
        m.lastTick = time.Time(msg)
        return m, tickCmd() // Keep ticking
    }
    return m, nil
}
```

### Loading States

```go
type model struct {
    loading bool
    spinner spinner.Model
}

func (m model) View() string {
    if m.loading {
        return fmt.Sprintf("%s Loading...", m.spinner.View())
    }
    return m.contentView()
}
```

---

## Terminal Size Handling

```go
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg := msg.(type) {
    case tea.WindowSizeMsg:
        m.width = msg.Width
        m.height = msg.Height

        // Resize components
        m.viewport.Width = msg.Width
        m.viewport.Height = msg.Height - 4
        m.list.SetSize(msg.Width, msg.Height)
    }
    return m, nil
}
```

---

## Color Themes

### ANSI Color Palette

```go
// Basic 16 colors (most compatible)
var (
    black   = lipgloss.Color("0")
    red     = lipgloss.Color("1")
    green   = lipgloss.Color("2")
    yellow  = lipgloss.Color("3")
    blue    = lipgloss.Color("4")
    magenta = lipgloss.Color("5")
    cyan    = lipgloss.Color("6")
    white   = lipgloss.Color("7")
    // Bright variants: 8-15
)

// 256 colors
var purple = lipgloss.Color("99")

// True color (hex)
var customPink = lipgloss.Color("#FF6B9D")
```

### Theme Structure

```go
type Theme struct {
    Primary    lipgloss.Color
    Secondary  lipgloss.Color
    Background lipgloss.Color
    Text       lipgloss.Color
    Muted      lipgloss.Color
    Success    lipgloss.Color
    Warning    lipgloss.Color
    Error      lipgloss.Color
}

var DarkTheme = Theme{
    Primary:    lipgloss.Color("205"),
    Secondary:  lipgloss.Color("39"),
    Background: lipgloss.Color("236"),
    Text:       lipgloss.Color("252"),
    Muted:      lipgloss.Color("241"),
    Success:    lipgloss.Color("82"),
    Warning:    lipgloss.Color("214"),
    Error:      lipgloss.Color("196"),
}
```

---

## Testing TUI Apps

### Model Testing

```go
func TestModel_Update(t *testing.T) {
    m := initialModel()

    // Simulate key press
    newModel, _ := m.Update(tea.KeyMsg{Type: tea.KeyDown})

    model := newModel.(model)
    if model.cursor != 1 {
        t.Errorf("expected cursor at 1, got %d", model.cursor)
    }
}
```

### View Testing

```go
func TestModel_View(t *testing.T) {
    m := model{items: []string{"one", "two"}, cursor: 0}

    view := m.View()

    if !strings.Contains(view, "> one") {
        t.Error("expected cursor on first item")
    }
}
```

---

## Other Frameworks

### Textual (Python)

```python
from textual.app import App, ComposeResult
from textual.widgets import Header, Footer, Static

class MyApp(App):
    BINDINGS = [("q", "quit", "Quit")]

    def compose(self) -> ComposeResult:
        yield Header()
        yield Static("Hello, World!")
        yield Footer()
```

### Ratatui (Rust)

```rust
fn ui(f: &mut Frame, app: &App) {
    let chunks = Layout::default()
        .direction(Direction::Vertical)
        .constraints([
            Constraint::Length(3),
            Constraint::Min(1),
            Constraint::Length(3),
        ])
        .split(f.size());

    let block = Block::default().borders(Borders::ALL).title("Header");
    f.render_widget(block, chunks[0]);
}
```

### Ink (React for Terminal)

```tsx
import React from 'react';
import {render, Box, Text} from 'ink';

const App = () => (
    <Box flexDirection="column">
        <Text color="green">Hello, World!</Text>
    </Box>
);

render(<App />);
```

---

## Performance Considerations

- **Minimize re-renders**: Only update changed portions of the view
- **Use viewport for large content**: Don't render off-screen content
- **Batch updates**: Combine multiple state changes before re-rendering
- **Profile with `pprof`**: Identify bottlenecks in update/view cycles
- **Cache computed styles**: Lip Gloss styles can be expensive to create

---

## Accessibility

- **High contrast colors**: Ensure text is readable
- **Keyboard-only navigation**: All actions accessible via keyboard
- **Screen reader considerations**: Terminal apps have limited screen reader support, but clear text helps
- **Status announcements**: Use clear text updates for state changes
