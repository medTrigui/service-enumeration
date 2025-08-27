# MkDocs Setup Guide

Complete guide for setting up, building, and deploying the Service Enumeration Cheat Sheets documentation.

## Prerequisites

### Required Software
- **Python 3.8+** - [Download](https://www.python.org/downloads/)
- **Git** - [Download](https://git-scm.com/downloads)
- **GitHub Account** - For hosting and deployment

### Verify Installation
```bash
# Check Python version
python --version

# Check pip
pip --version

# Check Git
git --version
```

## Local Development Setup

### 1. Clone the Repository
```bash
# Clone your fork or the main repository
git clone https://github.com/yourusername/service-enumeration.git
cd service-enumeration
```

### 2. Install Dependencies
```bash
# Install Python dependencies
pip install -r requirements.txt

# Alternative: Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 3. Test Local Development Server
```bash
# Start development server
mkdocs serve

# Access documentation at: http://127.0.0.1:8000
# Auto-reloads on file changes
```

### 4. Build Static Site
```bash
# Build production site
mkdocs build

# Output directory: ./site/
# Upload contents to any web server
```

## GitHub Pages Deployment

### Automatic Deployment (Recommended)

The repository includes GitHub Actions for automatic deployment:

1. **Enable GitHub Pages**
   - Go to repository Settings
   - Navigate to Pages section
   - Source: "GitHub Actions"

2. **Push to Main Branch**
   ```bash
   git add .
   git commit -m "Initial documentation setup"
   git push origin main
   ```

3. **Monitor Deployment**
   - Check Actions tab for deployment status
   - Site will be available at: `https://yourusername.github.io/service-enumeration`

### Manual Deployment
```bash
# Build and deploy manually
mkdocs gh-deploy

# This creates/updates the gh-pages branch
# GitHub Pages will serve from this branch
```

## Configuration Customization

### Update Repository Information
Edit `mkdocs.yml`:

```yaml
# Update these fields
site_name: Your Custom Title
site_url: https://yourusername.github.io/repository-name
repo_name: yourusername/repository-name
repo_url: https://github.com/yourusername/repository-name

# Update social links
extra:
  social:
    - icon: fontawesome/brands/github
      link: https://github.com/yourusername
    - icon: fontawesome/brands/twitter
      link: https://twitter.com/yourusername
```

### Custom Styling
Modify `docs/stylesheets/extra.css` for:
- Color scheme adjustments
- Font modifications
- Layout customizations
- Brand-specific styling

### Add Google Analytics (Optional)
```yaml
# In mkdocs.yml
extra:
  analytics:
    provider: google
    property: G-XXXXXXXXXX  # Your tracking ID
```

## Content Management

### Adding New Cheat Sheets

1. **Create Markdown File**
   ```bash
   # Add to appropriate category
   touch docs/database/postgresql.md
   touch docs/network/dns.md
   ```

2. **Update Navigation**
   ```yaml
   # In mkdocs.yml nav section
   nav:
     - Database Services:
       - PostgreSQL: database/postgresql.md
   ```

3. **Follow Content Standards**
   - Use consistent formatting
   - Include all methodology sections
   - Test all commands
   - Add security warnings

### Content Structure
```
docs/
├── index.md              # Homepage
├── quick-start.md         # Getting started guide
├── database/             
│   ├── index.md          # Category overview
│   ├── mysql.md          # Service cheat sheet
│   └── oracle-tns.md     # Service cheat sheet
├── mail/
├── network/
├── file/
├── resources/
│   ├── tools.md          # Tool reference
│   └── references.md     # Additional resources
└── contributing.md       # Contribution guide
```

## Advanced Configuration

### Custom Domain
1. **Add CNAME file**
   ```bash
   echo "your-domain.com" > docs/CNAME
   ```

2. **Configure DNS**
   - Add CNAME record pointing to: `yourusername.github.io`

3. **Update mkdocs.yml**
   ```yaml
   site_url: https://your-domain.com
   ```

### Enhanced Features

#### Search Enhancement
```yaml
# In mkdocs.yml
plugins:
  - search:
      lang: en
      separator: '[\s\-,:!=\[\]()"`/]+|\.(?!\d)|&[lg]t;|(?!\b)(?=[A-Z][a-z])'
```

#### Additional Plugins
```bash
# Install additional plugins
pip install mkdocs-awesome-pages-plugin
pip install mkdocs-git-revision-date-localized-plugin

# Add to mkdocs.yml
plugins:
  - awesome-pages
  - git-revision-date-localized
```

## Maintenance

### Regular Updates
```bash
# Update dependencies
pip install --upgrade -r requirements.txt

# Check for security vulnerabilities
pip audit

# Update MkDocs
pip install --upgrade mkdocs mkdocs-material
```

### Content Reviews
- **Monthly**: Review and update command accuracy
- **Quarterly**: Check for new tools and techniques
- **Annually**: Major version updates and restructuring

### Performance Monitoring
- **Build times**: Monitor CI/CD performance
- **Site speed**: Test page load times
- **Search functionality**: Verify search accuracy

## Troubleshooting

### Common Issues

#### Build Failures
```bash
# Check configuration syntax
mkdocs config

# Validate markdown files
mkdocs build --strict

# Clear cache and rebuild
rm -rf site/
mkdocs build
```

#### Missing Dependencies
```bash
# Reinstall all dependencies
pip uninstall -r requirements.txt -y
pip install -r requirements.txt
```

#### GitHub Actions Failures
1. Check Actions tab for error details
2. Verify permissions (Settings > Actions > General)
3. Ensure requirements.txt is up to date
4. Check for YAML syntax errors

### Performance Issues
```bash
# Build with verbose output
mkdocs build --verbose

# Check file sizes
du -sh docs/

# Optimize images
# Use compressed images and appropriate formats
```

### Content Issues
- **Broken links**: Use `mkdocs build --strict` to detect
- **Missing pages**: Check navigation configuration
- **Formatting**: Validate markdown syntax

## Best Practices

### Development Workflow
1. **Create feature branch** for changes
2. **Test locally** with `mkdocs serve`
3. **Build successfully** with `mkdocs build`
4. **Submit pull request** for review
5. **Deploy automatically** via Actions

### Content Guidelines
- **Test all commands** before publishing
- **Include security warnings** for dangerous operations
- **Maintain consistent formatting**
- **Update regularly** with new techniques

### SEO Optimization
- **Descriptive page titles**
- **Meta descriptions** in frontmatter
- **Proper heading structure**
- **Internal linking**

## Support

### Getting Help
- **Issues**: Create GitHub issues for bugs
- **Discussions**: Use GitHub Discussions for questions
- **Documentation**: Reference MkDocs and Material theme docs

### Resources
- **MkDocs**: https://www.mkdocs.org/
- **Material Theme**: https://squidfunk.github.io/mkdocs-material/
- **GitHub Pages**: https://docs.github.com/en/pages
- **Markdown Guide**: https://www.markdownguide.org/

### Community
- **Contributing**: See CONTRIBUTING.md
- **Code of Conduct**: Professional and respectful interaction
- **Recognition**: Contributors acknowledged in documentation
