# Automatic Index File Generation Using Shell and Find

This document explains a shell command that automatically generates `index.md` files for every directory in a repository. The purpose of this approach is to enable smooth navigation in static site generators such as GitHub Pages and Jekyll, where directory listing is not supported by default.

---

## The Problem This Command Solves

GitHub Pages does not behave like a traditional file server. When a directory is opened in a browser, its contents are not displayed unless an `index.html` or `index.md` file exists. As repositories grow large and deeply nested, manually creating index files becomes inefficient and error-prone.

The command described in this document programmatically scans the entire repository and creates an `index.md` file in every directory. Each index file contains a list of links to the directory’s immediate files and subdirectories, enabling structured navigation through the content.

---

## The Complete Command

The following command is executed from the root of the repository and operates recursively on all directories except the Git metadata directory.

```bash
find . -type d ! -path "./.git*" -exec sh -c '
  dir="{}"
  echo "# Index of $dir" > "$dir/index.md"
  for f in "$dir"/*; do
    name=$(basename "$f")
    echo "- [$name]($name)" >> "$dir/index.md"
  done
' \
```

Understanding the find Command
The find utility is a standard Unix tool used to search for files and directories based on specified criteria. In this case, the search begins from the current directory, denoted by the dot (.), which represents the repository root.

The option -type d restricts the search results to directories only. This ensures that the command operates on folders and not individual files.

The expression ! -path "./.git*" explicitly excludes the .git directory. The .git directory contains internal Git metadata and must never be modified, as doing so can corrupt the repository.

Executing a Shell Command for Each Directory
The -exec sh -c '...' portion instructs find to execute a shell command for each directory it discovers. The shell used here is sh, which ensures portability across Unix-like systems.

The placeholder {} is automatically replaced by the current directory path during each execution. This allows the script to dynamically operate on every directory found.

Assigning the Current Directory to a Variable
Inside the shell block, the line below assigns the current directory path to a variable named dir.

```sh
dir="{}"
```

Using a variable improves readability and avoids repeating the directory path multiple times within the script.

Creating or Overwriting the Index File
The next command writes a heading into an index.md file within the current directory.

```sh
echo "# Index of $dir" > "$dir/index.md"
```

The echo command outputs a markdown header containing the directory name. The greater-than symbol (>) redirects the output into the index.md file. If the file already exists, it is overwritten, ensuring the index always reflects the latest directory contents.

Iterating Over Directory Contents
A for loop is used to iterate over every file and subdirectory inside the current directory.

```sh
for f in "$dir"/*; do
```

This loop processes only immediate children of the directory and does not recurse further. This behavior ensures that each index file represents a single directory level, keeping navigation clean and predictable.

Extracting File and Directory Names
Within the loop, the basename utility is used to extract the name of each file or directory from its full path.

```sh
name=$(basename "$f")
```

The basename command strips the directory path and returns only the final component, which is suitable for use in markdown links.

Writing Markdown Links
For each file or directory, a markdown list entry is appended to the index file.

```sh
echo "- [$name]($name)" >> "$dir/index.md"
```

The double greater-than symbol (>>) appends content rather than overwriting it. Each entry is written in markdown link format, where both the link text and the link target are the same name. This format allows static site generators such as Jekyll to resolve the links correctly.

Command Termination and Escaping
The trailing backslash at the end of the command escapes the newline, ensuring that the find command interprets the entire script as a single execution block.

```bash
' \
```

This is required for correct parsing by the shell when multiline commands are used.

Resulting Directory Structure Behavior
After running this command, every directory in the repository contains an index.md file that lists its contents. When combined with GitHub Pages and Jekyll, this structure allows users to click into directories and navigate the repository naturally through rendered markdown pages.

Conclusion
This command provides a scalable and automated solution for generating navigable index pages in large repositories. By leveraging standard Unix tools such as find, sh, basename, and shell redirection, it transforms a static file hierarchy into a structured documentation site without introducing external dependencies. This approach aligns well with professional DevOps workflows and static documentation best practices.
