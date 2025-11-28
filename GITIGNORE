# 0L1 Labs .gitignore

# ==========================================
# Node.js / JavaScript / Frontend
# ==========================================

# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
package-lock.json
yarn.lock
pnpm-lock.yaml

# Production builds
/build
/dist
/.next
/out

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Testing
/coverage
/.nyc_output
*.lcov

# Cache
.cache
.parcel-cache
.eslintcache

# Misc
.DS_Store
*.swp
*.swo
*~
.vscode/
.idea/

# ==========================================
# Rust / Solana / Anchor
# ==========================================

# Rust build artifacts
/target
**/*.rs.bk
Cargo.lock

# Anchor
.anchor
.DS_Store
target/
**/*.rs.bk
node_modules
test-ledger/
.yarn

# Solana
.solana/
test-ledger/
.solana-keygen/

# ==========================================
# Zero-Knowledge Circuits (Circom)
# ==========================================

# Circom build artifacts
*.r1cs
*.wasm
*.sym
*.zkey
*.vkey
circuit.json
verification_key.json
witness.wtns
proof.json
public.json

# Compiled circuits
build/
circuits/build/
circuits/out/

# Powers of Tau files (large ceremony files)
pot*.ptau
powersOfTau*.ptau

# ==========================================
# Python (if used for tooling)
# ==========================================

# Byte-compiled / optimized
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
pip-wheel-metadata/
*.egg-info/
.installed.cfg
*.egg

# ==========================================
# Documentation
# ==========================================

# Auto-generated docs
docs/_build/
docs/.doctrees/
site/

# ==========================================
# IDE / Editor
# ==========================================

# VSCode
.vscode/*
!.vscode/settings.json.example
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json

# JetBrains
.idea/
*.iml
*.iws
*.ipr
out/

# Sublime Text
*.sublime-project
*.sublime-workspace

# Vim
[._]*.s[a-v][a-z]
[._]*.sw[a-p]
[._]s[a-rt-v][a-z]
[._]ss[a-gi-z]
[._]sw[a-p]

# Emacs
*~
\#*\#
/.emacs.desktop
/.emacs.desktop.lock
*.elc
auto-save-list
tramp
.\#*

# ==========================================
# OS
# ==========================================

# macOS
.DS_Store
.AppleDouble
.LSOverride
Icon
._*
.Spotlight-V100
.Trashes

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/

# Linux
.directory
.Trash-*

# ==========================================
# Secrets & Keys
# ==========================================

# Private keys (NEVER commit these!)
*.pem
*.key
*.p12
*.pfx
*.cer
*.crt
id_rsa*
id_dsa*
id_ecdsa*
id_ed25519*

# Wallet files
*.wallet
*.json.key
keypair.json

# Environment secrets
.env
.env.local
.env.*.local
secrets.json
config.secret.*

# ==========================================
# Logs
# ==========================================

logs/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
lerna-debug.log*
.npm
.eslintcache

# ==========================================
# Testing
# ==========================================

# Test results
coverage/
*.lcov
.nyc_output/
test-results/
test-reports/

# ==========================================
# Database
# ==========================================

*.db
*.sqlite
*.sqlite3

# ==========================================
# Temporary Files
# ==========================================

tmp/
temp/
*.tmp
*.bak
*.backup
*.old

# ==========================================
# API & Backend (when added)
# ==========================================

# Environment
.env
.env.test
.env.production

# Database
*.db
*.sqlite
*.sqlite3

# Session storage
sessions/

# Uploads
uploads/
public/uploads/

# ==========================================
# Docker (if used)
# ==========================================

# Docker
*.dockerignore
docker-compose.override.yml

# ==========================================
# Analytics & Monitoring
# ==========================================

# Sentry
.sentryclirc

# ==========================================
# Package Manager Locks (Optional)
# ==========================================

# Uncomment these if you want to commit lock files
# package-lock.json
# yarn.lock
# Cargo.lock

# ==========================================
# Custom 0L1 Labs Specific
# ==========================================

# Circuit test vectors
test-vectors/
# Large ceremony files
ceremonies/
# Generated proofs (testing)
proofs/test_*
# Local blockchain data
.solana-local/
validator-data/
