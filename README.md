---
description: https://gitmastery.me/ practices and learnings note
---

# 👨‍💻 Git Basic to Advance

## Git mastery Learning Notes

1. To Create a new repository for project - `git init`

The hidden .git directory now contains all the information Git needs.

2. To check the current state at any time - `git status`

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

3.  "When you join an existing project, you don't start from scratch. Instead, you clone the remote repository, which creates a complete copy on your machine—including all the code, history, and branches."

    "Think of it like checking out a book from the library, except you get the entire library's records too! Use `git clone` to get started."

{% code overflow="wrap" %}
```
git clone url
```
{% endcode %}

3. When you modify files, you need to explicitly tell Git which changes should be included in the next commit. This is called 'staging' and works with `git add`."

{% code overflow="wrap" %}
```bash
git add .  // Add all files to the staging area
```
{% endcode %}

4. A commit is like a snapshot of your project at a specific point in time. Each commit needs a message that describes what was changed. This is important for traceability.

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

5. When you want to remove files that are tracked by Git, you should use `git rm` rather than just deleting them manually. This ensures Git properly tracks the deletion.

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

6. To Display all the existing branches -&#x20;

{% code overflow="wrap" %}
```bash
git branch
```
{% endcode %}

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

7. Git introduced the `git switch` command to make branch operations clearer. Use `git switch -c feature` to create and switch to the new branch in one step. This is the preferred modern way instead of the older `git checkout -b`."

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

8. You can switch to any existing branch using `git switch` . This is much clearer than the old `git checkout` which could be confusing because it did many different things.

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

9. **Checkout** can do many things - switch branches, restore files, and more. That's why Git introduced **switch** and **restore** - to make intentions clearer.

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

10. This is the modern way in Git. The -c flag stands for 'create' and does exactly the same as the older 'git checkout -b', but it's clearer and more intuitive.

> The switch -c pattern is the modern, recommended method for creating and switching branches. It was introduced in Git 2.23 to separate branch operations from other checkout functions and make them more intuitive.

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

11. The first step is to add a connection to the remote repository using `git remote add`. **This doesn't transfer any code yet—it just creates the connection**.

> Remote repositories are central to collaborative development workflows. Most Git-based systems like GitHub, GitLab, and Bitbucket work by hosting remote repositories that team members connect to.

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

12. `git push` uploads your COMMITS, not individual files! You must make a commit before you can push. Your local commits only exist on your computer until you push them.

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

> The difference between local and remote repository is fundamental: Local commits only exist on your machine. Only through git push do they become visible to your team. This means: You can make as many local commits as you want and then push them all at once!

<figure><img src=".gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

13. When pushing a branch for the first time, you should use the -u (or --set-upstream) option. This **links your local branch with the remote branch**, making future pushes and pulls easier.

> In professional teams, new features are typically developed on separate branches and then pushed for review before being merged into the main codebase. This is a central part of the pull request workflow.

<figure><img src=".gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

***

#### <mark style="color:$warning;">Git Advance</mark>

14. In professional teams, we never merge directly into main. First feature → develop for testing, then develop → main for production.

<figure><img src=".gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

For example: Merge the 'feature/user-auth' branch into the 'develop' branch

<figure><img src=".gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

15. **main** is our production branch. Only tested, stable code goes in here. That's why we tested on develop first!

<figure><img src=".gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

For example: Merge the 'develop' branch into the 'main' branch

<figure><img src=".gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

16. Sometimes merges don't go as planned. When the same part of a file has been changed differently in both branches, a merge conflict occurs.

You have two options: Either you resolve the conflict manually, or you abort the merge with `git merge --abort` and prepare better.

> Git Flow Workflow 🌊
>
> 📦 main: Production-ready code
>
> 🔧 develop: Integration and testing
>
> ✨ feature/\*: New features
>
> This workflow prevents untested code from reaching production. Many teams also use release branches!

<figure><img src=".gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

**The Complete Workflow:**

{% code overflow="wrap" %}
```
1. Create a feature branch from main: git switch -c feature/user-auth
2. Make changes to files and stage them with git add
3. Commit changes with descriptive messages
4. Push your branch to remote: git push origin feature/user-auth
5. Switch back to main: git switch main
6. Merge the feature: git merge feature/user-auth
```
{% endcode %}

**Pull Request Workflow:**

{% code overflow="wrap" %}
```
1. You push your feature branch to the remote repository
2. On GitHub/GitLab, you open a Pull Request from feature/user-auth to main
3. Your teammates receive a notification
4. They review your code, leave comments, and suggest improvements
5. You make changes based on feedback and push again
6. Once approved, someone merges the PR into main
7. Your feature is now part of the main codebase!
```
{% endcode %}

> Feature branch workflow is the industry standard. Developers create isolated branches, push them to remote repos (GitHub/GitLab), create Pull Requests for code review, and merge after approval. This collaborative approach prevents unstable code from reaching production and improves code quality through peer review.

***

{% code overflow="wrap" %}
```
Start by creating a feature branch: 'git switch -c feature/user-auth'
Modify the auth.js file, then use 'git add' to stage your changes
Commit with: 'git commit'
Push to remote: 'git push origin feature/user-auth'
Switch back to main: 'git switch main'
Finally merge: 'git merge feature/user-auth'
```
{% endcode %}

#### <mark style="color:orange;">Some Scenarios</mark>

<mark style="color:orange;">**Story 01**</mark>

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Task: Master the hotfix workflow for emergency production fixes.

{% code overflow="wrap" %}
```bash
1. git checkout -c hotfix/critical-security-patch
2. git add .
3. git commit -m "hotfix marge"
4. git switch main
5. git merge hotfix/critical-security-patch    
```
{% endcode %}

<mark style="color:orange;">**Story 02**</mark>

<figure><img src=".gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Task: Learn the professional release workflow: branch, prepare, merge, and tag. This is how teams ship stable software to production.

> Release branches are used in Git Flow to prepare production releases. They allow final bug fixes and documentation updates without blocking ongoing development. The release is tagged for easy reference and rollback if needed.

{% code overflow="wrap" %}
```bash
1. git switch -c release/2.0.0
2. git add .
3. git commit -m "Prepare the release 2.0.0"
4. git switch main
5. git merge release/2.0.0
6. git tag v2.0.0
```
{% endcode %}

<mark style="color:orange;">**Story 03**</mark>

<figure><img src=".gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Task: Learn the fundamentals of team-based Git workflow and make your first collaborative contribution.

{% code overflow="wrap" %}
```bash
1. git pull origin main
2. git switch -c feature/team-profile
3. nano team.md ( te edit team name)
4. git add . 
5. git commit -m "Add my profile as team list"
6. git push origin feature/team-profile
```
{% endcode %}

<mark style="color:orange;">**Story 04**</mark>

<figure><img src=".gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Task: Master merge conflict resolution to become a confident team collaborator.

{% code overflow="wrap" %}
```bash
1. git add .
2. git commit -m "local changes"
3. git pull origin main
4. nano src/auth/login.js (then remove the conflict head ====)
5. git commit -m "resolve marge conflict"
```
{% endcode %}
