# Git Exercises with Git Merge Examples

Here’s a beginner-friendly guide with added examples for using Git merge.

---

### **Setup and Basics**

#### **Exercise 1: Git Configuration**
1. Set up your Git username and email. (this step can be skipped if you already have Git setup, go to Step 2 to verify)
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "your_email@example.com"
    ```
2. Verify your configuration.
    ```bash
    git config --list
    ```

---

#### **Exercise 2: Initialize a Repository**
1. Create a new directory and initialize a repository.
    ```bash
    mkdir git-merge-exercises
    cd git-merge-exercises
    git init
    ```

---

### **Working with Files**

#### **Exercise 3: Adding and Committing Files**
1. Create a file and commit it.
    ```bash
    echo "Hello, Git!" > hello.txt
    git add hello.txt
    git commit -m "Add hello.txt with initial content"
    ```

---

### **Branching and Merging**

#### **Exercise 4: Merging Fast-Forward**
1. Create and switch to a new branch.
    ```bash
    git checkout -b feature-fast-forward
    ```
2. Add a new file and commit it.
    ```bash
    echo "This is a fast-forward merge example." > fast-forward.txt
    git add fast-forward.txt
    git commit -m "Add fast-forward.txt"
    ```
3. Switch back to the `main` branch.
    ```bash
    git checkout main
    ```
4. Merge the `feature-fast-forward` branch into `main`.
    ```bash
    git merge feature-fast-forward
    ```
5. Verify the merge by checking the files in the `main` branch.
    ```bash
    ls
    ```

---

#### **Exercise 5: Merging with Divergence**
1. Create and switch to a new branch.
    ```bash
    git checkout -b feature-divergent
    ```
2. Modify `hello.txt` and commit the change.
    ```bash
    echo "This is a change in the feature-divergent branch." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in feature-divergent branch"
    ```
3. Switch back to the `main` branch and make a different modification to `hello.txt`.
    ```bash
    echo "This is a change in the main branch." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in main branch"
    ```
4. Merge the `feature-divergent` branch into `main`.
    ```bash
    git merge feature-divergent
    ```
5. **Resolve the conflict**:
    - Open `hello.txt`, and you will see conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`).
    - Edit the file to resolve the conflict by keeping the desired content.
    - After resolving the conflict, stage the file.
      ```bash
      git add hello.txt
      ```
    - Complete the merge.
      ```bash
      git commit -m "Resolve merge conflict between main and feature-divergent"
      ```

---

#### **Exercise 6: Merging Without Conflicts**
1. Create and switch to another new branch.
    ```bash
    git checkout -b feature-no-conflict
    ```
2. Create a new file and commit it.
    ```bash
    echo "This file will not conflict." > no-conflict.txt
    git add no-conflict.txt
    git commit -m "Add no-conflict.txt"
    ```
3. Switch back to the `main` branch and merge the `feature-no-conflict` branch.
    ```bash
    git merge feature-no-conflict
    ```
4. Verify the merge by checking the files in the `main` branch.
    ```bash
    ls
    ```

---

#### **Exercise 7: Merge with Rebase Comparison**
1. Create and switch to a branch.
    ```bash
    git checkout -b feature-rebase
    ```
2. Modify `hello.txt` and commit the change.
    ```bash
    echo "This is a rebase example." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in feature-rebase branch"
    ```
3. Switch back to the `main` branch and make another change.
    ```bash
    echo "This is another change in main." >> hello.txt
    git add hello.txt
    git commit -m "Update hello.txt in main branch"
    ```
4. Rebase the `feature-rebase` branch onto `main`.
    ```bash
    git checkout feature-rebase
    git rebase main
    ```
5. Switch back to `main` and merge the rebased branch.
    ```bash
    git checkout main
    git merge feature-rebase
    ```
