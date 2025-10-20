Do

```
config nu
```

Then paste the contents into it.
# Windows

```nu

# config.nu
#
# Installed by:
# version = "0.102.0"
#
# This file is used to override default Nushell settings, define
# (or import) custom commands, or run any other startup tasks.
# See https://www.nushell.sh/book/configuration.html
#
# This file is loaded after env.nu and before login.nu
#
# You can open this file in your default editor using:
# config nu
#
# See `help config nu` for more options
#
# You can remove these comments if you want or leave
# them for future reference.

$env.config.show_banner = false
$env.VISUAL = 'code'
$env.EDITOR = 'hx'

zoxide init nushell | save -f ~/.zoxide.nu
source ~/.zoxide.nu

# 
# The Following alias expects a couple of dependencies
# fzf, ripgrep, vscode, and chromium/edge checkout

# Set your preferred editor for opening files

# Function for efficient fzf usage
def f [$query] { fzf -q $query }
def fc [$query] { fzf -q $query --preview "type {}" | str trim | ^($env.EDITOR) $in }
def fbranch [] { git branch | fzf | str trim | git checkout  $in }
def dbranch [] { git branch | fzf | str trim | git branch -D $in }
def fcd [$query] { fzf -q $query --preview "dir {}" | str trim | ^($env.EDITOR) $in }

# Combine ripgrep with fzf to search and open files in huge codebases
def frg [$query] { rg $query | fzf }
def rcpp [$query] { rg -tcpp $query | fzf }
def rjs [$query] { rg -tjs $query | fzf }

# Small git-based productivity functions
alias branch = git branch
alias checkout = git checkout
alias status = git status 
alias ga = git add 
alias pull = git branch --show-current | git pull origin $in
alias gc = git commit -m
alias git_reset = git reset --hard HEAD
alias clean = git clean -fd
alias log = git log --oneline --decorate -n 5


let CR_SRC = "Q:/cr/src"

alias e = z $CR_SRC
alias eod = z ($CR_SRC | path join "out" "debug_x64")
alias eor = z ($CR_SRC | path join "out" "release_x64")
alias pdf = z ($CR_SRC | path join "third_party" "microsoft_pdf" "msinternal")


alias ocode = code $CR_SRC
alias ohx = hx $CR_SRC
alias onvim = nvim $CR_SRC

alias ac = autoninja chrome
alias aclow = autoninja chrome -j 300
alias a = autoninja 

def gen_clangd [] { vpython3 ($EDGE_DIR | path join tools\clang\scripts\generate_compdb.py) -p . -o ($CR_SRC | path join "compile_commands.json") }

alias format = git cl format

# let COMPILER_PATH = $CR_SRC + "\\third_party\\llvm-build\\Release+Asserts\\bin\\clang-cl.exe"

# let CLANGD_PATH = $CR_SRC + "\\third_party\\llvm-build\\Release+Asserts\\bin\\clangd.exe"

# let CLANG_FORMAT_PATH = $CR_SRC + "\\buildtools\\win\\clang-format.exe"

alias chrome =  chrome --user-data-dir=c:\data\dir

def kill_cr [] { 
    let parent_pid = $nu.pid
    ps | where ppid == $nu.pid | where name == "chrome.exe" | each {|e| kill -f -q $e.pid}

}

def update_main [] { 
    git checkout main; 
    git pull origin main; 
    gclient sync -D 
} 

def rebase_branch [] {
    git fetch origin main;
    git rebase FETCH_HEAD;
    gclient sync -D;
}

def rebuild [$type] {
    if $type == "r" {
        autogn x64 release;
        eor;
    } else if $type == "d" {
        autogn x64 debug;
        eod;
    } else {
        echo "Invalid build type. Use 'r' for release and 'd' for debug";
        return;
    }
    ac;
}

def reset_profile [] { rmdir /s /q c:\data\dir }

echo "Dev environment initialized"

```