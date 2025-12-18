# UV Package Manager - Comprehensive Installation Plan

## Executive Summary

**UV** is an extremely fast Python package and project manager written in Rust, developed by Astral (creators of Ruff). It's designed to replace pip, pip-tools, pipx, poetry, pyenv, virtualenv, and more - offering **10-100x faster** performance than pip while maintaining full compatibility.

This document provides a complete migration plan for switching from pip to UV for this Azure AI project.

---

## 1. What is UV?

### Overview
UV is a next-generation Python package manager that provides:
- **Speed**: 10-100x faster than pip due to Rust implementation
- **Compatibility**: Drop-in replacement for pip with familiar CLI
- **Unified Tool**: Replaces multiple tools (pip, pip-tools, virtualenv, poetry, etc.)
- **Global Cache**: Disk-space efficient with dependency deduplication
- **Python Management**: Can install and manage Python versions
- **Modern Features**: Universal lockfiles, workspaces, and more

### Key Advantages Over pip

| Feature | pip | UV |
|---------|-----|-----|
| **Speed** | Baseline | 10-100x faster |
| **Dependency Resolution** | Basic | Advanced with PubGrub algorithm |
| **Cache System** | Limited | Global cache with deduplication |
| **Virtual Environments** | Requires virtualenv | Built-in `uv venv` |
| **Lockfiles** | Requires pip-tools | Native support with `uv pip compile` |
| **Python Version Management** | Requires pyenv | Built-in `uv python` |
| **Parallel Downloads** | No | Yes |
| **Platform Support** | Python-based | Native binaries (no Python required) |

---

## 2. Current Project Analysis

### Project Dependencies (from requirements.txt)
```
requests==2.32.5
python-dotenv==1.2.1
openai==2.8.0
pandas==2.3.3
azure-cosmos==4.14.2
semantic-kernel==1.37.0
azure-ai-projects==1.1.0b4
azure-ai-agents==1.2.0b6
azure-ai-inference==1.0.0b9
azure-ai-evaluation[redteam]==1.13.7
azure-identity==1.25.1
opentelemetry-sdk==1.38.0
azure-core-tracing-opentelemetry==1.0.0b12
azure-monitor-opentelemetry==1.8.2
opentelemetry-instrumentation-openai-v2==2.1b0
azure-search-documents==11.6.0
fastapi==0.121.3
uvicorn[standard]==0.38.0
orjson==3.11.4
numpy==2.2.6
a2a-sdk[sqlite]==0.3.12
sse-starlette==3.0.3
starlette==0.50.0
mcp==1.21.2
httpx==0.28.1
fastmcp==2.13.1
```

**Total Dependencies**: 26 packages with pinned versions
**Special Considerations**: 
- Multiple Azure SDK packages (beta versions)
- FastAPI/Uvicorn for web services
- OpenTelemetry for observability
- MCP (Model Context Protocol) packages

---

## 3. UV Installation for Windows

### Method 1: PowerShell Standalone Installer (Recommended)
```powershell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**Advantages**:
- No Python required
- Native binaries
- Self-updating capability
- Fastest method

### Method 2: Windows Package Manager (WinGet)
```cmd
winget install --id=astral-sh.uv -e
```

**Advantages**:
- Integrated with Windows package management
- Easy updates via WinGet

### Method 3: Scoop
```cmd
scoop install main/uv
```

### Method 4: pip/pipx (If Python Already Installed)
```cmd
pip install uv
# OR
pipx install uv
```

**Note**: Methods 1-3 are preferred as they don't require Python to be installed.

### Verify Installation
```cmd
uv --version
```

Expected output: `uv 0.9.18` (or newer)

---

## 4. Step-by-Step Migration Plan

### Phase 1: Install UV (Choose one method from Section 3)

**Recommended Command for Windows**:
```cmd
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

### Phase 2: Backup Current Environment

```cmd
REM Navigate to project directory
cd c:\Code\learn\TechWorkshop-L300-AI-Apps-and-agents\src

REM Backup current requirements (if needed)
copy requirements.txt requirements.txt.backup
```

### Phase 3: Using UV with Existing Virtual Environment

**Option A: Use Existing venv with UV**
```cmd
REM Activate your existing virtual environment first
..\venv\Scripts\activate

REM Install dependencies using UV (much faster than pip)
uv pip install -r requirements.txt
```

**Option B: Create Fresh Virtual Environment with UV**
```cmd
REM Navigate to project root
cd c:\Code\learn\TechWorkshop-L300-AI-Apps-and-agents

REM Create new virtual environment using UV
uv venv

REM Activate the new environment
.venv\Scripts\activate

REM Install all dependencies
uv pip install -r src\requirements.txt
```

### Phase 4: Sync Dependencies (Recommended Method)

**Using `uv pip sync`** ensures the environment exactly matches requirements.txt:
```cmd
REM Activate environment
.venv\Scripts\activate

REM Sync environment (removes packages not in requirements.txt)
uv pip sync src\requirements.txt
```

**Difference between `uv pip install` and `uv pip sync`**:
- `uv pip install`: Installs packages but doesn't remove extras
- `uv pip sync`: Ensures environment exactly matches the lockfile (removes unlisted packages)

---

## 5. UV Command Reference

### Essential Commands for This Project

#### Virtual Environment Management
```cmd
REM Create virtual environment
uv venv

REM Create with specific Python version
uv venv --python 3.11

REM Remove virtual environment
rmdir /s /q .venv
```

#### Package Installation
```cmd
REM Install from requirements.txt
uv pip install -r requirements.txt

REM Sync environment to match requirements.txt exactly
uv pip sync requirements.txt

REM Install a single package
uv pip install <package-name>

REM Install with extras (e.g., FastAPI with standard extras)
uv pip install "uvicorn[standard]"
```

#### Dependency Locking
```cmd
REM Compile requirements.txt (if you have requirements.in)
uv pip compile requirements.in -o requirements.txt

REM Upgrade all dependencies
uv pip compile requirements.in -o requirements.txt --upgrade

REM Upgrade specific package
uv pip compile requirements.in -o requirements.txt --upgrade-package openai
```

#### Inspection
```cmd
REM List installed packages
uv pip list

REM Show package details
uv pip show <package-name>

REM Check for outdated packages
uv pip list --outdated
```

---

## 6. Workflow Comparison: pip vs UV

### Current pip Workflow
```cmd
REM Create virtual environment
python -m venv venv
venv\Scripts\activate

REM Install dependencies (slow)
pip install -r requirements.txt

REM Update packages
pip install --upgrade <package>
pip freeze > requirements.txt
```
**Time**: ~2-5 minutes for initial install

### New UV Workflow
```cmd
REM Create virtual environment
uv venv

REM Activate
.venv\Scripts\activate

REM Sync dependencies (fast)
uv pip sync requirements.txt

REM Update packages
uv pip install --upgrade <package>
uv pip freeze > requirements.txt
```
**Time**: ~10-30 seconds for initial install (10-100x faster)

---

## 7. Azure AI Project Considerations

### Compatibility with Azure SDKs
✅ **UV is fully compatible with Azure SDK packages**, including:
- Beta versions (e.g., `azure-ai-projects==1.1.0b4`)
- Packages with extras (e.g., `azure-ai-evaluation[redteam]`)
- Complex dependencies like OpenTelemetry

### Special Notes for This Project

1. **Beta Packages**: UV handles beta versions correctly (e.g., `1.1.0b4`)
2. **Extras Syntax**: UV supports extras like `uvicorn[standard]` and `a2a-sdk[sqlite]`
3. **FastAPI/Uvicorn**: No issues with web framework packages
4. **MCP Packages**: Model Context Protocol packages work seamlessly
5. **OpenTelemetry**: All instrumentation packages are compatible

### Testing Recommendations
After migration, verify critical functionality:
```cmd
REM Test imports
python -c "import azure.ai.projects; print('Azure AI Projects OK')"
python -c "import semantic_kernel; print('Semantic Kernel OK')"
python -c "import fastapi; print('FastAPI OK')"
python -c "import mcp; print('MCP OK')"

REM Run your application
python chat_app.py
```

---

## 8. Advanced Features (Optional)

### Project-Based Management (Modern Approach)

Instead of managing `requirements.txt`, UV supports modern project management:

#### Convert to pyproject.toml (Optional)
```cmd
REM Initialize UV project
uv init

REM Add dependencies from requirements.txt
uv add requests==2.32.5 python-dotenv==1.2.1 openai==2.8.0
REM ... continue for all packages
```

This creates:
- `pyproject.toml`: Project metadata and dependencies
- `uv.lock`: Universal lockfile (cross-platform)
- `.venv`: Virtual environment

**Benefits**:
- Modern Python standard (PEP 621)
- Better dependency management
- Cross-platform reproducibility
- Automatic lock file generation

### Using UV with Existing requirements.txt (Recommended for Now)

**Stay with requirements.txt** if:
- Team is familiar with pip workflow
- Don't want to change project structure
- Need gradual migration

**UV works perfectly with requirements.txt** using the pip interface.

---

## 9. Performance Benchmarks

Based on official UV benchmarks for similar projects:

| Operation | pip | UV | Speedup |
|-----------|-----|-----|---------|
| Initial install (cold cache) | 60s | 3s | 20x |
| Install (warm cache) | 45s | 0.5s | 90x |
| Resolve dependencies | 15s | 0.3s | 50x |
| Create virtual environment | 3s | 0.1s | 30x |

**Expected time savings for this project**:
- Initial setup: 2-3 minutes → 10-20 seconds
- Daily dependency updates: 30-60 seconds → 1-2 seconds
- CI/CD pipeline: Significant reduction in build times

---

## 10. Gotchas and Caveats

### Known Issues
1. **First Run**: Initial download of packages to cache may not be dramatically faster
2. **Warm Cache**: Speed improvements are most noticeable with populated cache
3. **Git Dependencies**: If using packages from git, ensure proper authentication
4. **Build Tools**: Some packages requiring compilation still need build tools

### Troubleshooting

#### Issue: UV not found after installation
```cmd
REM Restart terminal or add to PATH manually
set PATH=%USERPROFILE%\.local\bin;%PATH%
```

#### Issue: Package installation fails
```cmd
REM Use verbose mode to diagnose
uv pip install -r requirements.txt -v

REM Clear cache if needed
uv cache clean
```

#### Issue: Virtual environment activation issues
```cmd
REM Ensure you're using the correct activation script
.venv\Scripts\activate
REM NOT: .venv\bin\activate (that's for Unix)
```

---

## 11. Recommended Migration Path

### Conservative Approach (Recommended)

1. **Install UV** alongside pip
2. **Test UV** with existing requirements.txt
3. **Use UV for installations** but keep requirements.txt format
4. **Monitor performance** and verify compatibility
5. **Optional**: Migrate to pyproject.toml later

### Commands for Conservative Migration
```cmd
REM Step 1: Install UV
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"

REM Step 2: Navigate to project
cd c:\Code\learn\TechWorkshop-L300-AI-Apps-and-agents

REM Step 3: Create new environment with UV
uv venv

REM Step 4: Activate environment
.venv\Scripts\activate

REM Step 5: Install dependencies with UV
uv pip sync src\requirements.txt

REM Step 6: Verify installation
python -c "import azure.ai.projects; import fastapi; import mcp; print('All imports successful')"

REM Step 7: Test application
cd src
python chat_app.py
```

---

## 12. Next Steps and Best Practices

### Immediate Actions
1. ✅ Install UV using PowerShell installer
2. ✅ Create test virtual environment with UV
3. ✅ Install dependencies using `uv pip sync`
4. ✅ Verify application functionality
5. ✅ Document any issues encountered

### Long-term Improvements
1. **Add to CI/CD**: Update GitHub Actions/Azure Pipelines to use UV
2. **Team Training**: Ensure team members understand UV commands
3. **Documentation**: Update project README with UV instructions
4. **Consider pyproject.toml**: Migrate to modern project structure

### Updating Dependencies
```cmd
REM Check for outdated packages
uv pip list --outdated

REM Update specific package
uv pip install --upgrade azure-ai-projects

REM Regenerate requirements.txt
uv pip freeze > requirements.txt
```

---

## 13. Additional Resources

- **Official Documentation**: https://docs.astral.sh/uv/
- **GitHub Repository**: https://github.com/astral-sh/uv
- **Migration Guide**: https://docs.astral.sh/uv/guides/migration/
- **pip Compatibility**: https://docs.astral.sh/uv/pip/compatibility/
- **Benchmarks**: https://github.com/astral-sh/uv/blob/main/BENCHMARKS.md

---

## 14. Summary

### Why UV?
- **10-100x faster** than pip
- **Drop-in replacement** - no workflow changes needed
- **Better dependency resolution**
- **Global cache** saves disk space
- **Modern tooling** with backward compatibility

### Migration Complexity
- ⭐⭐⭐⭐⭐ **Very Easy** (5/5)
- No code changes required
- Works with existing requirements.txt
- Can revert to pip anytime

### Recommended Action
**Install UV and use it alongside pip initially**. The performance benefits are substantial with virtually no risk or downtime.

```cmd
REM Quick Start - Copy and paste these commands
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
cd c:\Code\learn\TechWorkshop-L300-AI-Apps-and-agents
uv venv
.venv\Scripts\activate
uv pip sync src\requirements.txt
```

---

**Report Generated**: December 17, 2025  
**UV Version**: 0.9.18  
**Project**: TechWorkshop-L300-AI-Apps-and-agents  
**Platform**: Windows with cmd.exe shell
