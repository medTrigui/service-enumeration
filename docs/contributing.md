# Contributing

Thank you for your interest in contributing to the Service Enumeration Cheat Sheets project. Your contributions help make these resources more comprehensive and valuable for the cybersecurity community.

## Ways to Contribute

### Content Contributions
- **New techniques and commands**
- **Updated tool usage**
- **Additional service coverage**
- **Real-world examples**
- **Security considerations**

### Documentation Improvements
- **Clarity enhancements**
- **Error corrections**
- **Formatting improvements**
- **Missing information**
- **Outdated references**

### Technical Contributions
- **Website improvements**
- **CI/CD enhancements**
- **Testing automation**
- **Performance optimizations**

## Contribution Process

### 1. Fork the Repository
```bash
# Fork on GitHub, then clone your fork
git clone https://github.com/yourusername/service-enumeration.git
cd service-enumeration
```

### 2. Create a Feature Branch
```bash
# Create and switch to feature branch
git checkout -b feature/your-feature-name

# Examples:
git checkout -b feature/mysql-udf-exploitation
git checkout -b fix/smtp-enum-commands
git checkout -b docs/improve-snmp-section
```

### 3. Make Your Changes
- Follow the existing format and style
- Test all commands before submitting
- Include proper explanations
- Add relevant security warnings

### 4. Test Your Changes
```bash
# Test MkDocs build locally
pip install -r requirements.txt
mkdocs serve

# Access local site at http://127.0.0.1:8000
```

### 5. Commit Your Changes
```bash
# Add and commit changes
git add .
git commit -m "Add MySQL UDF exploitation techniques"

# Use descriptive commit messages:
# - "Add new SNMP enumeration commands"
# - "Fix typos in Oracle TNS section"
# - "Update SMB authentication methods"
```

### 6. Push and Create Pull Request
```bash
# Push to your fork
git push origin feature/your-feature-name
```

Then create a Pull Request on GitHub with:
- Clear description of changes
- Reason for the contribution
- Testing performed
- Any relevant screenshots

## Content Guidelines

### Command Accuracy
- **Test all commands** before submitting
- **Provide context** for when commands apply
- **Include expected output** when helpful
- **Note version dependencies** if applicable

### Documentation Standards
- **Clear explanations** for complex procedures
- **Consistent formatting** with existing content
- **Proper markdown syntax**
- **Professional language**

### Security Considerations
- **Authorization reminders** for testing commands
- **Risk assessments** for dangerous operations
- **Legal disclaimers** where appropriate
- **Ethical guidelines** emphasis

## File Structure

### Cheat Sheet Format
Each service cheat sheet should follow this structure:

```markdown
# Service Name

Brief description of the service and its purpose.

## Service Information
- **Default Ports:** List common ports
- **Protocol:** TCP/UDP specification
- **Common Implementations:** Software versions

## Discovery
Port scanning and service detection commands.

## Enumeration
Information gathering techniques and tools.

## Authentication
Credential testing and brute force methods.

## Exploitation
Attack vectors and exploitation techniques.

## Post-Exploitation
Persistence and privilege escalation methods.

## Defensive Considerations
Security hardening and detection methods.
```

### Code Block Standards
```bash
# Good: Clear command with explanation
# MySQL version enumeration
nmap -p 3306 --script mysql-info target

# Good: Multiple related commands
mysql -h target -u root -p
mysql> SELECT version();
mysql> SHOW DATABASES;

# Avoid: Commands without context
nmap target
mysql
```

### Command Categories
Organize commands by purpose:
- **Discovery:** Finding and identifying services
- **Enumeration:** Gathering information
- **Authentication:** Testing credentials
- **Exploitation:** Gaining access
- **Post-Exploitation:** Maintaining access

## Review Process

### Automated Checks
Pull requests trigger automated checks for:
- MkDocs build success
- Markdown syntax validation
- Link verification
- Spelling and grammar

### Manual Review
Maintainers review submissions for:
- **Technical accuracy**
- **Content quality**
- **Formatting consistency**
- **Security considerations**
- **Legal compliance**

### Feedback and Iteration
- Address reviewer feedback promptly
- Be open to suggestions and improvements
- Collaborate on refinements
- Learn from the review process

## Recognition

### Contributors
All contributors are recognized in:
- **Git commit history**
- **GitHub contributors list**
- **Documentation credits**
- **Release notes**

### Attribution
When contributing content from other sources:
- **Provide proper attribution**
- **Verify licensing compatibility**
- **Respect intellectual property**
- **Add value through adaptation**

## Community Standards

### Code of Conduct
- **Be respectful** and professional
- **Welcome newcomers** and help them learn
- **Provide constructive feedback**
- **Focus on content quality**

### Communication
- **Use clear, descriptive language**
- **Ask questions when unclear**
- **Share knowledge generously**
- **Acknowledge others' contributions**

## Technical Requirements

### Development Environment
```bash
# Install Python dependencies
pip install -r requirements.txt

# Serve documentation locally
mkdocs serve

# Build static site
mkdocs build
```

### Testing Checklist
Before submitting:
- [ ] MkDocs builds without errors
- [ ] All links work correctly
- [ ] Commands are tested and accurate
- [ ] Formatting is consistent
- [ ] No spelling/grammar errors

## Getting Help

### Documentation Issues
- **Create GitHub issues** for problems
- **Use descriptive titles** and details
- **Provide context** and examples
- **Tag appropriately** (bug, enhancement, question)

### Technical Questions
- **Check existing issues** first
- **Search documentation** for answers
- **Ask in discussions** for guidance
- **Contact maintainers** for complex issues

### Resources
- **MkDocs Documentation:** https://www.mkdocs.org/
- **Material Theme:** https://squidfunk.github.io/mkdocs-material/
- **Markdown Guide:** https://www.markdownguide.org/
- **GitHub Flow:** https://guides.github.com/introduction/flow/

Thank you for contributing to the cybersecurity community through these cheat sheets. Your expertise and effort help protect organizations and advance security knowledge.
