#### Commit Message Guidelines

Proper commit messages are crucial for clear project history and traceability. Here's how to format them:

1. **Commit Message Tags**:
   - **[CLEANUP]**: Use this tag for commits that involve code cleanup without changing the behavior. Examples include renaming variables, editing comments, and removing unused imports.
     ```
     git commit -m "[CLEANUP] Removed obsolete comments and renamed variables for clarity"
     ```
   - **[BUGFIX/FEATURE][DOPS 123456]**: For commits that fix bugs or add features linked to a specific DOPS task. Always include the DOPS ID.
     ```
     git commit -m "[BUGFIX][DOPS 123456] Do not try to schedule prepare jobs at every scheduler run"
     ```
   - **[Revert]**: Use when reverting a commit. This should be used sparingly and typically under exceptional circumstances.
     ```
     git commit -m "[Revert] Revert '[BUGFIX][DOPS 123456] Fixed NPE in ReplacementHandler'"
     ```

3. **Extended Commit Messages**:
   - For detailed explanations, use a multi-line commit message, created by using `-m` in sequence:
     ```sh
     git commit -m "[BUGFIX][DOPS 123456] Do not try to schedule prepare jobs at every scheduler run" -m "- Fixed null pointer exception when calling methodCallBla. Press ENTER before closing the quotes to add a line break. Repeat as needed.Then close the quotes and hit ENTER twice to apply the commit."
     ```
