# Shared Slides Repository

This is a library of shared slide content and preprocessing tools for creating educational presentations. The repository provides markdown-based slide content organized by topic (Git, CI, Java, build systems, etc.) and Ruby-based tooling for content generation and PDF creation.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Essential Dependencies
Install the following dependencies before working with this repository:
- `sudo apt update && sudo apt install -y ruby hugo poppler-utils inotify-tools google-chrome-stable`
- Ruby 3.2+ (repository specifies 3.4.5 in `.ruby-version` but 3.2+ works)
- Hugo (for serving content in parent repositories)
- Chrome/Chromium (for PDF generation)
- poppler-utils (provides `pdfinfo` for PDF validation)
- inotify-tools (provides `inotifywait` for file watching)

### Basic Validation
- Check Ruby syntax: `ruby -c preprocess.rb && ruby -c makepdfs.rb` - takes <1 second
- Test preprocessing: `./preprocess.rb` - takes <1 second, processes any `*generator.md` files found
- Check dependencies: `hugo version && pdfinfo -v && which inotifywait && google-chrome-stable --version`

### Core Functionality
- **Content preprocessing**: `./preprocess.rb` - processes markdown files with generator patterns
  - Looks for files ending in `*generator.md`
  - Replaces content between `<!-- write-here "path/to/file.md" -->` and `<!-- end-write -->` comments
  - Creates `_index.md` files in the same directory
  - **NEVER CANCEL**: Always completes in <1 second, but set timeout to 30+ seconds for safety
- **PDF generation**: `./makepdfs.rb ROOT_DIR` - generates PDFs from HTML slides using headless Chrome
  - Requires a Git repository with remote URL configured
  - Requires built Hugo site with HTML files accessible via HTTP
  - **NEVER CANCEL**: Can take 5 seconds to 5 minutes per presentation. Set timeout to 30+ minutes
  - Uses Chrome in headless mode with specific flags for consistent rendering
  - Automatically detects slide vs document format and saves appropriately
- **Content serving**: `./serve.sh` - serves content with Hugo and watches for changes
  - **IMPORTANT**: Must be run from a parent directory containing both `content/` and `shared-slides/`
  - Will fail if run directly from this repository (no Hugo configuration present)

## Usage Patterns

### This Repository Is A Content Library
- **NOT a standalone Hugo site** - provides shared content for other repositories
- **Intended usage**: Include as a submodule or subdirectory in course/presentation repositories
- **Tested with**: DanySK/Course-DTM-SE-3, unibo-spe/spe-slides, DanySK/Course-Laboratory-of-Software-Systems, apice-at-disi/oop-lab

### Typical Workflow in Parent Repositories
1. Include this repository as `shared-slides/` subdirectory
2. Create `content/*/generator.md` files with include directives
3. Run `shared-slides/preprocess.rb` to inject content
4. Use Hugo to build and serve the complete site
5. Optionally generate PDFs with `shared-slides/makepdfs.rb`

### Example Content Integration
```markdown
---
title: Git Basics
---

<!-- write-here "shared-slides/git/intro.md" -->
<!-- end-write -->

<!-- write-here-code "shared-slides/git/dvcs-concepts.md" -->
<!-- end-write -->
```

After running `shared-slides/preprocess.rb`, this becomes a complete slide with injected content. The `write-here-code` variant excludes the warning comment.

## Validation

### Always Test Your Changes
- **Preprocessing validation**: Create a test scenario with generator files and verify content injection works
- **Syntax validation**: Run `ruby -c *.rb` to check Ruby syntax
- **Link validation**: GitHub Actions will run markdown link checking automatically
- **Integration testing**: Test with one of the example repositories from the GitHub workflow

### Manual Testing Scenarios
```bash
# Test preprocessing workflow
mkdir -p /tmp/test/content/example
echo '---
title: Test
---
<!-- write-here "shared-slides/git/intro.md" -->
<!-- end-write -->' > /tmp/test/content/example/generator.md
cd /tmp/test && cp -r [repo-path] shared-slides
shared-slides/preprocess.rb
# Verify _index.md was created with injected content

# Test no-warning variant
echo '<!-- write-here-code "shared-slides/ci/core-concepts.md" -->
<!-- end-write -->' > /tmp/test/content/example/generator.md
shared-slides/preprocess.rb
# Verify _index.md has no warning comment

# Test PDF generation (requires Git repo and HTML files)
git init && git remote add origin https://github.com/test/test
echo '<html><body><h1>Test</h1></body></html>' > index.html
shared-slides/makepdfs.rb .
# Check for generated PDF files
```

### Expected Timings
- **Preprocessing**: <1 second for typical repositories (measured: 0.06s)
- **PDF generation**: 5-15 seconds per simple page, up to 5 minutes for complex presentations
- **Hugo serving**: Starts in 1-2 seconds
- **Link checking**: 30 seconds to 2 minutes depending on content
- **NEVER CANCEL**: All operations complete quickly but always set timeouts of 60+ seconds for builds and 30+ seconds for PDF generation

## Repository Structure

### Content Directories
- `git/` - Git and version control concepts (14 files)
- `ci/` - Continuous Integration topics
- `java/` - Java programming content
- `build-systems/` - Build tools and dependency management
- `devops/` - DevOps practices and containerization
- `containerization/` - Docker and container technologies

### Key Scripts
- `preprocess.rb` - Content injection processor (main tool)
- `makepdfs.rb` - PDF generation from HTML slides
- `serve.sh` - Hugo development server with file watching
- `.ruby-version` - Specifies Ruby 3.4.5 (but 3.2+ works)

### Configuration Files
- `.gitmodules` - Git submodule configuration
- `renovate.json` - Dependency update automation
- `.mergify.yml` - Automated PR merging rules

## Common Tasks

### Repository Statistics
- **Total markdown files**: 28
- **Repository size**: ~548KB
- **Content topics**: 6 main categories
- **Primary language**: Ruby + Markdown

### Working with Content
- **Add new slides**: Create markdown files in appropriate topic directories
- **Test content**: Always run preprocessing to verify include patterns work
- **Validate links**: GitHub Actions automatically checks all markdown links
- **Update dependencies**: Renovate automatically manages updates

### Troubleshooting
- **serve.sh fails**: Ensure you're running from a directory with Hugo configuration and `content/` folder
- **PDF generation fails**: Check Chrome installation and HTML content accessibility
- **Preprocessing fails**: Verify file paths in include directives are correct
- **Ruby version issues**: Repository specifies 3.4.5 but 3.2+ is sufficient for all functionality

## CI/CD Information
- **GitHub Actions**: Runs Ruby syntax checking and preprocessing tests
- **Test repositories**: Validates against multiple real-world course repositories
- **Link checking**: Automated validation of all markdown links
- **No build artifacts**: Repository contains only source content and scripts