# Siren: Mermaid Integration for LaTeX via Rust

Siren is a LaTeX package that allows you to embed Mermaid.js diagrams directly into your TeX documents using a Rust-based toolchain. It utilizes the `selkie` binary for layout generation and `librsvg` for high-quality vector output, bypassing the need for heavy Node.js or Headless Chrome dependencies.

## Key Features

- **Native Mermaid DSL**: Write diagrams directly within LaTeX environments using standard Mermaid syntax.
- **Rust-Powered Performance**: Fast rendering via the `selkie` utility (built on Rust).
- **MD5 Caching**: Automatically skips re-rendering if the diagram source code is unchanged, speeding up document compilation.
- **Vector Graphics**: Generates PDF output for sharp, scalable diagrams that integrate perfectly with LaTeX typography.
- **Inline Error Reporting**: Captures toolchain stderr and displays it directly in the document if a build fails, with helpful troubleshooting guidance.
- **60-Second Timeout**: Prevents hanging builds if the toolchain encounters issues.
- **Cross-Platform Support**: Works on macOS, Linux, and Windows (with proper PATH configuration).

---

## Prerequisites & Dependencies

Siren requires two external binaries to be available in your system's `PATH`.

### 1. selkie

The Rust binary that parses Mermaid code and generates SVG layouts.

- **Installation**: Requires [Cargo](https://doc.rust-lang.org/cargo/).

```bash
cargo install selkie
```

- **Verify installation**: Run `selkie --help` to ensure it's accessible from your terminal.

### 2. librsvg (rsvg-convert)

A high-performance library used to convert SVG files into PDF vectors for LaTeX inclusion.

- **macOS**: `brew install librsvg`
- **Linux (Ubuntu/Debian)**: `sudo apt install librsvg2-bin`
- **Linux (Fedora/RHEL)**: `sudo dnf install librsvg2-tools`
- **Windows**: 
  - Via Chocolatey: `choco install librsvg`
  - Via MSYS2: Install through MSYS2 package manager
  - Via manual download: Add the `bin` directory to your PATH

- **Verify installation**: Run `rsvg-convert --version` to ensure it's accessible.

### 3. LaTeX Environment

Requires a modern distribution (TeX Live 2023+, MiKTeX 23.1+) and the following packages:

- `fancyvrb` (verbatim text handling)
- `shellesc` (shell escape support)
- `graphicx` (image inclusion)
- `xcolor` (color support for error messages)
- `pdftexcmds` (engine-compatible MD5 hashing)

Most modern LaTeX distributions include these packages by default.

---

## Quick Start

1. **Install prerequisites** (see above)
2. **Compile with shell escape**:

```bash
pdflatex --shell-escape main.tex
```

3. **Run twice** to resolve cross-references (standard LaTeX practice).

---

## Usage

### Compilation Requirement

You **must** compile your document with the `--shell-escape` flag to allow LaTeX to execute external binaries. Without this flag, Siren will not function.

**Command Line:**
```bash
# Recommended approach
pdflatex --shell-escape main.tex

# Or with xelatex
xelatex --shell-escape main.tex

# Or with lualatex
lualatex --shell-escape main.tex
```

**Using .latexmkrc (Recommended for Common Editors):**

To avoid typing `--shell-escape` every time, create a `.latexmkrc` file in your project directory:

```perl
# File: .latexmkrc
$latex = 'latex --shell-escape %O %S';
$pdflatex = 'pdflatex --shell-escape %O %S';
$lualatex = 'lualatex --shell-escape %O %S';
$xelatex = 'xelatex --shell-escape %O %S';
```

This configuration works with common editors like Zed, VS Code, Vim, Emacs, and TeXShop. After creating the file, you can simply run `latexmk -pdf main.tex` or `latexmk` and the shell escape flag will be automatically applied.

**Editor Configuration:**

Many editors allow you to configure the `--shell-escape` flag in their settings:
- **VS Code** (LaTeX Workshop): Set `"latex-workshop.latex.cmd": "pdflatex -shell-escape %f%"`
- **Vim** (vimtex): Add `let g:vimtex_latexmk_options = '-shell-escape'`
- **Emacs** (AUCTeX): Set `TeX-PDF-mode` with shell escape enabled

### The `mermaid` Environment

The `mermaid` environment accepts an optional argument for `includegraphics` options (e.g., `width`, `height`).

```latex
\documentclass{article}
\usepackage{siren}

\begin{document}

\section{Flow Diagram Example}

\begin{mermaid}[width=0.8\textwidth]
graph LR
    A[Source] --> B{Siren}
    B --> C[SVG]
    C --> D[PDF Vector]
    D --> E[Final Output]
    
    style A fill:#e1f5fe
    style B fill:#fff3e0
    style C fill:#f3e5f5
    style D fill:#e8f5e9
\end{mermaid}

\end{document}
```

**Supported Mermaid Diagram Types:**
- Flowcharts (`graph`, `flowchart`)
- Sequence diagrams
- Class diagrams
- State diagrams
- Gantt charts
- Git graphs
- Entity relationship diagrams
- User journey diagrams

### Adding Captions and Labels

The `mermaid` environment now behaves like `\includegraphics` — it does **not** automatically center or float. To add captions, labels, and control placement, wrap the `mermaid` environment inside a `figure` environment and use `\centering`:

```latex
\documentclass{article}
\usepackage{siren}

\begin{document}

\section{System Architecture}

\begin{figure}[htbp]
\centering
\begin{mermaid}[width=0.9\linewidth]
sequenceDiagram
    participant U as User
    participant FE as Frontend/App
    participant API as Backend API
    participant DB as Database
    participant Email as Email Service

    U->>FE: Clicks "Sign Up"
    activate FE
    
    FE->>API: POST /api/signup {email, password, username}
    activate API
    
    API->>DB: Check if email already exists
    DB-->>API: Result (not found)
    
    API->>DB: INSERT new user
    DB-->>API: User created (userId)
    
    API->>Email: Send welcome + verification email
    activate Email
    Email-->>API: Email queued/sent
    deactivate Email
    
    API-->>FE: 201 Created + JWT token
    deactivate API
    
    FE-->>U: Show "Check your email" screen
    deactivate FE

    Note over U,Email: Later...

    U->>FE: Clicks verification link from email
    FE->>API: GET /api/verify?token=xxx
    API->>DB: Mark user as verified
    DB-->>API: OK
    API-->>FE: 200 OK + redirect to login
    FE-->>U: Redirects to login page
\end{mermaid}
\caption{Architecture Overview}
\label{fig:architecture}
\end{figure}

This is our `\ref{fig:architecture}` showing the core services.

---

## Behavior Notes

The `mermaid` environment is intentionally lightweight and does not add automatic centering, floating, or caption support. This design mirrors how `\includegraphics` works, giving you full control over layout:

- **No automatic centering**: Use `\centering` inside a `figure` environment or wrap in `\begin{center}...\end{center}`
- **No floating**: To enable floats (placement control with `[htbp]`), wrap in a `figure` environment
- **No captions**: Use `\caption{...}` inside a `figure` environment
- **No labels**: Use `\label{...}` inside a `figure` environment for `\ref` and `\autoref` support

This approach follows LaTeX best practices and gives you maximum flexibility for positioning your diagrams.

\end{document}
```

---

## Technical Workflow

Siren follows a four-phase workflow to render Mermaid diagrams:

1. **Extraction**: LaTeX writes the environment body to a unique `.mmd` file (e.g., `siren_out_1.mmd`).

2. **Checksum**: The package calculates an MD5 hash of the `.mmd` file using `pdftexcmds` for engine compatibility (works with pdfTeX, XeTeX, and LuaTeX).

3. **Execution** (if hash differs from cache):
   - Existing `.pdf` and `.svg` files are deleted to prevent stale data
   - `selkie` converts `.mmd` to `.svg` (all output redirected to log)
   - `rsvg-convert` converts `.svg` to `.pdf` (appended to same log)
   - Current MD5 hash is saved to `.md5` cache file
   - A 60-second timeout prevents hanging builds

4. **Inclusion**: 
   - If `.pdf` exists: Inserted via `\includegraphics`
   - If `.pdf` missing: Display formatted error box with toolchain log

**File Naming Pattern:**
All temporary files use the prefix `siren_out_` followed by the diagram counter:
- `siren_out_1.mmd` - Source Mermaid code
- `siren_out_1.svg` - Intermediate SVG output
- `siren_out_1.pdf` - Final PDF for LaTeX
- `siren_out_1.log` - Toolchain output (stdout + stderr)
- `siren_out_1.md5` - Cache checksum

---

## Caching Behavior

Siren uses MD5-based caching to optimize compilation times:

- **Cache Hit**: If the `.mmd` file hasn't changed, Siren skips the toolchain entirely and reuses the existing `.pdf` output.
- **Cache Miss**: If the `.mmd` file changes (or cache is missing), Siren re-renders the diagram.
- **Cache Validation**: On each run, the MD5 hash of the `.mmd` file is compared to the cached value.

**Benefits:**
- Significantly faster compilation for unchanged diagrams
- Only modified diagrams are re-rendered
- Cache is automatically invalidated when diagram content changes

**Manual Cache Clear:**
If you need to force re-rendering, delete the `.md5` cache file for the affected diagram.

---

## Error Handling

### Error Box Display

When the toolchain fails to generate a PDF, Siren displays a formatted error box:

```
┌─────────────────────────────────────────────────────────────────┐
│ ⚠️ SIren Toolchain Failure [Diagram 1]                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│ Toolchain Output:                                              │
│ ┌────────────────────────────────────────────────────────────┐ │
│ │ Error: Failed to parse diagram                             │ │
│ │   --> input.mmd:5:20                                       │ │
│ │    |                                                     │ │
│ │  5 | graph LR                                             │ │
│ │    |                    ^ unexpected token                 │ │
│ └────────────────────────────────────────────────────────────┘ │
│                                                                  │
│ Trace:                                                           │
│ Input: siren_out_1.mmd                                           │
│ Expected: siren_out_1.pdf                                        │
│ Status: FAILED                                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Error Types

| Error | Cause | Solution |
|-------|-------|----------|
| Empty or old image | Diagram failed to render; old artifact persists | Check the red error box for specific syntax errors |
| `"Command not found"` | `selkie` or `rsvg-convert` not in PATH | Verify installations with `selkie --help` and `rsvg-convert --version` |
| `MD5 Undefined` | `pdftexcmds` package missing or incompatible | Ensure `pdftexcmds` is installed; use updated TeX distribution |
| Runaway argument | Verbatim content not properly terminated | Check for unmatched `\begin{mermaid}` / `\end{mermaid}` |
| Timeout error | Diagram too complex or toolchain hung | Check for infinite loops in Mermaid syntax; optimize diagram |
| Permission denied | Cannot write to working directory | Check file permissions; ensure write access to output folder |
| Shell escape disabled | Compilation without `--shell-escape` | Recompile with `pdflatex --shell-escape main.tex` |

### Debugging Tips

1. **Check the log**: If the error box shows "No log found," verify:
   - `--shell-escape` flag was used
   - Working directory is writable
   - Shell escape is enabled in your editor settings

2. **Manual verification**: Run commands manually to test the toolchain:
   ```bash
   # Extract .mmd from LaTeX output
   cat siren_out_1.mmd
   
   # Test selkie independently
   selkie siren_out_1.mmd -o test.svg
   
   # Test rsvg-convert independently
   rsvg-convert -f pdf -o test.pdf test.svg
   ```

3. **Check PATH**: Ensure binaries are in your PATH:
   ```bash
   which selkie
   which rsvg-convert
   ```

---

## Advanced Configuration

### Custom Binary Paths (Windows)

If `selkie` or `rsvg-convert` are not in your system PATH, you can specify absolute paths by modifying the `siren.sty` file:

```latex
% In siren.sty, replace:
% \ShellEscape{selkie "\siren@mmd" ...}

% With:
\ShellEscape{C:\\path\\to\\selkie.exe "\siren@mmd" ...}
```



---

## Examples

### Flowchart with Styling

```latex
\begin{mermaid}[width=0.7\textwidth]
graph TD
    A[Start] --> B{Valid Input?}
    B -->|Yes| C[Process Data]
    B -->|No| D[Show Error]
    C --> E[Generate Output]
    D --> F[Exit]
    E --> F
    
    style A fill:#c8e6c9,stroke:#2e7d32
    style B fill:#fff9c4,stroke:#f9a825
    style C fill:#bbdefb,stroke:#1565c0
    style D fill:#ffcdd2,stroke:#c62828
\end{mermaid}
```

### Sequence Diagram

```latex
\begin{mermaid}[width=\textwidth]
sequenceDiagram
    participant User
    participant API
    participant DB
    
    User->>API: Request data
    API->>DB: Query database
    DB-->>API: Return results
    API-->>User: JSON response
\end{mermaid}
```

### Class Diagram

```latex
\begin{mermaid}[width=0.8\textwidth]
classDiagram
    class Diagram~{render(), validate()}
    class Renderer~{renderSVG(), renderPDF()}
    class Cache~{check(), update()}
    
    Diagram --> Renderer
    Diagram --> Cache
    Renderer "1" --> "1" Cache
\end{mermaid}
```

---

## Troubleshooting

### Common Issues

#### Diagram Not Rendering

**Symptoms**: Blank space where diagram should be, or error box appears.

**Solutions**:
1. Verify `--shell-escape` flag is used during compilation
2. Check that `selkie` and `rsvg-convert` are in your PATH
3. Look at the error log for syntax errors in Mermaid code
4. Test Mermaid syntax independently using [Mermaid Live Editor](https://mermaid.live/)

#### Slow Compilation

**Symptoms**: Document takes much longer than expected to compile.

**Solutions**:
1. Ensure caching is working (check `\typeout` messages in log)
2. Only modify changed diagrams (Siren should skip unchanged ones)
3. Simplify complex diagrams if possible
4. Wrap diagrams in `figure` environment for better float control

#### Path Issues on Windows

**Symptoms**: "Command not found" errors for `selkie` or `rsvg-convert`.

**Solutions**:
1. Add binary locations to system PATH
2. Restart your terminal/editor after PATH changes
3. Use absolute paths in `siren.sty` as a workaround
4. Consider using WSL for better Unix tool integration

### Getting Help

If you encounter issues not covered here:

1. **Check the log file**: `siren_out_X.log` contains detailed toolchain output
2. **Verify installations**: Run `selkie --version` and `rsvg-convert --version`
3. **Test manually**: Try running commands manually from the terminal
4. **Check TeX distribution**: Ensure you have a recent version (TeX Live 2023+)
5. **Review Mermaid syntax**: Validate diagrams on [mermaid.live](https://mermaid.live)

---

## License

This LaTeX package is provided as-is for educational and research purposes.

---

## Credits

- **selkie**: Rust-based Mermaid renderer ([selkie GitHub](https://github.com/btucker/selkie))
- **librsvg**: SVG rendering library ([librsvg GitHub](https://github.com/GNOME/librsvg))
- **Mermaid**: JavaScript-based diagramming tool ([Mermaid.js](https://mermaid.js.org/))

---

*Last updated: 2026-03-18*
