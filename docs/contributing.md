# Contributing

Help improve these cheat sheets for the cybersecurity community.

## How to Contribute

- **New techniques** and exploitation methods
- **Updated commands** and tool usage
- **Bug fixes** and error corrections
- **Additional services** coverage

## Quick Process

1. **Fork** the repository on GitHub
2. **Create branch**: `git checkout -b feature/your-feature`
3. **Make changes** and test commands
4. **Commit**: `git commit -m "Add new technique"`  
5. **Push**: `git push origin feature/your-feature`
6. **Create Pull Request** with description

## Guidelines

### Command Standards
- **Test all commands** before submitting
- **Add clear explanations** and context
- **Follow existing format** and style
- **Include security warnings** where needed

### Cheat Sheet Structure
```markdown
# Service Name
## Service Information (ports, protocol)
## Discovery (port scanning)
## Enumeration (information gathering)
## Authentication (credential testing)
## Exploitation (attack vectors)
## Post-Exploitation (persistence)
```

## Testing

Before submitting:
```bash
# Install dependencies
pip install -r requirements.txt

# Test build
mkdocs serve
```

Check:
- [ ] Commands tested and working
- [ ] MkDocs builds without errors  
- [ ] Formatting consistent
- [ ] Links work correctly

## Recognition

Contributors are recognized in Git history and documentation credits.

## Community Standards

- Be respectful and professional
- Test all commands before submitting
- Follow established formatting
- Focus on accuracy and clarity

Thank you for helping improve cybersecurity knowledge sharing!
