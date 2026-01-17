# CTF Writeups & Security Blog

A Hugo-based blog template optimized for CTF writeups and cybersecurity content.

## Features

- **Syntax Highlighting**: Code blocks with line numbers and copy buttons
- **CTF Taxonomies**: Organized by categories, tags, and CTF events
- **Search Functionality**: Full-text search across all writeups
- **Responsive Design**: Mobile-friendly PaperMod theme
- **Dark/Light Mode**: Toggle between themes
- **Table of Contents**: Auto-generated TOC for each writeup
- **Reading Time**: Estimated reading time for articles

## Quick Start

### Creating a New Writeup

Use Hugo's archetype system to create a new writeup:

```bash
hugo new writeups/ctf-name-challenge-name.md
```

This will create a new writeup from the template at `archetypes/writeups.md` with all the necessary front matter.

### Front Matter Fields

```yaml
---
title: "Challenge Name"
date: 2024-01-15T10:00:00+07:00
draft: false  # Set to false when ready to publish
description: "Brief description"
categories: ["Web"]  # Choose from: Web, Pwn, Crypto, Reverse Engineering, Forensics, Misc, OSINT, Hardware
tags: ["SQL Injection", "Python"]  # Specific techniques/tools
ctfs: ["HackTheBox 2024"]  # CTF event name
author: "Zicc0Nguyen"
ShowToc: true
TocOpen: true
---
```

### Available Categories

- **Web**: Web exploitation challenges
- **Pwn**: Binary exploitation and pwn challenges
- **Crypto**: Cryptography challenges
- **Reverse Engineering**: RE and malware analysis
- **Forensics**: Digital forensics and steganography
- **Misc**: Miscellaneous challenges
- **OSINT**: Open-source intelligence
- **Hardware**: Hardware hacking challenges

### Building the Site

```bash
# Run local development server
hugo server -D

# Build for production
hugo --minify
```

### Deployment

The site uses GitHub Actions for automatic deployment to GitHub Pages. Push to the `main` branch to trigger deployment.

## File Structure

```
.
├── archetypes/
│   └── writeups.md          # Template for new writeups
├── content/
│   ├── writeups/            # CTF writeup posts
│   │   ├── _index.md
│   │   └── sample-web-challenge.md
│   └── search.md            # Search page
├── themes/
│   └── PaperMod/            # Hugo theme (submodule)
├── hugo.toml                # Site configuration
└── README.md
```

## Configuration Highlights

### Syntax Highlighting

The site uses Monokai color scheme with line numbers enabled. Supported languages include:

- Python, Bash, JavaScript, Go, C, C++
- SQL, PHP, Ruby, Rust
- Assembly, PowerShell
- And many more...

### Navigation Menu

- **Writeups**: Browse all writeups
- **Categories**: Browse by challenge category
- **Tags**: Browse by techniques/tools
- **Search**: Full-text search

## Tips for Writing Writeups

1. **Use descriptive titles**: Include CTF name and challenge category
2. **Add code blocks with language tags**: ```python, ```bash, ```sql
3. **Include screenshots**: Store in `static/images/` and reference with `![alt](/images/file.png)`
4. **Document your process**: Show reconnaissance, analysis, and exploitation steps
5. **Add key takeaways**: Help others learn from your solution
6. **Reference tools**: List all tools and resources used

## Example Writeup Structure

```markdown
## Challenge Information
[CTF details, category, points]

## Description
[Original challenge description]

## Reconnaissance
[Initial analysis and information gathering]

## Vulnerability Analysis
[Identify the vulnerability or challenge mechanism]

## Exploitation
[Step-by-step solution with code/commands]

## Flag
[The captured flag]

## Key Takeaways
[Lessons learned]

## Tools Used
[List of tools]

## References
[External resources]
```

## Customization

### Update Site Information

Edit `hugo.toml`:

```toml
title = 'Your Site Title'
[params]
  author = "Your Name"
  description = "Your description"
```

### Update Social Links

Edit the `params.socialIcons` section in `hugo.toml`:

```toml
[[params.socialIcons]]
  name = "github"
  url = "https://github.com/yourusername"
```

## Resources

- [Hugo Documentation](https://gohugo.io/documentation/)
- [PaperMod Theme Guide](https://github.com/adityatelange/hugo-PaperMod/wiki)
- [Markdown Cheat Sheet](https://www.markdownguide.org/cheat-sheet/)

## License

This template is free to use for your CTF writeups and security blog.
