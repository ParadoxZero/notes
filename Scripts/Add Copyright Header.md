```python
"""
Copyright (c) 2024 Sidhin S Thomas <sidhin.thomas@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
"""

import os
import sys
import re

# Define the copyright text
copyright_text = \
"""/*
 * BudgetBug - Budgeting and Expense Tracker with WebUI and API server
 * Copyright (C) 2025  Sidhin S Thomas <sidhin.thomas@gmail.com>
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU Affero General Public License as published
 * by the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU Affero General Public License for more details.
 *
 * You should have received a copy of the GNU Affero General Public License
 * along with this program.  If not, see <https://www.gnu.org/licenses/>.
 *
 * The source is available at: https://github.com/ParadoxZero/budgetbud
 */
"""

file_extensions = ['.tsx', '.ts']
regex_header = r"^\s*/\*([\s\S]*?)\*/"

default_ignore_patterns = [
    '.git/',
    '.vscode/',
    'node_modules/',
    'build/',
    'dist/',
    'coverage/',
    'package-lock.json',
]

def read_gitignore(directory):
    gitignore_path = os.path.join(directory, '.gitignore')
    if not os.path.exists(gitignore_path):
        return set(default_ignore_patterns)

    with open(gitignore_path, 'r') as file:
        lines = file.readlines()

    ignore_patterns = [line.strip() for line in lines if line.strip() and not line.startswith('#')]
    return set(default_ignore_patterns + ignore_patterns)

def should_ignore(path, ignore_patterns):
    for pattern in ignore_patterns:
        if pattern.endswith('/'):
            if path.startswith(pattern[:-1]):
                return True
        elif pattern in path:
            return True
    return False

def add_copyright_to_file(file_path):
    with open(file_path, 'r+') as file:
        content = file.read()
        if not re.match(regex_header, content):
            print(f"Adding copyright to {file_path}")
            file.seek(0, 0)
            file.write(copyright_text + '\n' + content)

def add_copyright_to_files(directory):
    ignore_patterns = read_gitignore(directory)
    for root, _, files in os.walk(directory):
        if should_ignore(root, ignore_patterns):
            continue
        for file in files:
            if any(file.endswith(ext) for ext in file_extensions):
                file_path = os.path.join(root, file)
                if not should_ignore(file_path, ignore_patterns):
                    add_copyright_to_file(file_path)

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python add_copyright.py <path_to_repository>")
        sys.exit(1)

    directory = sys.argv[1]
    add_copyright_to_files(directory)
    print(f"Copyright attribution added to all {file_extensions} files.")
```