#Intro to Git Branching

#### Why Use Git Branches

Because we will often have two (or many more) versions of the same code, one version that is live and another version that we develop on, git branches allow us to work on code before we make it live. It helps us avoid sending bad code into production!

<a href="http://nvie.com/img/2009/12/Screen-shot-2009-12-24-at-11.32.03.png"><img height="530px" src="http://nvie.com/img/2009/12/Screen-shot-2009-12-24-at-11.32.03.png" /></a><br />
<small><i>from [http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/)</i></small>

####Common Commands

* `git branch` – See all your branches
* `git checkout master` – Check out of the current branch and check into master branch
	* `git checkout -b [new branch name]` – Create a new branch and check into it
* `git merge [branch]` – Merge a branch into current branch
* `git push origin [new branch name]` – Push branch and commits to Github
* `git branch -d [branch name]` – Deletes branch from local system
	* `git push origin :[branch name]` – Deletes the branch on Github

#### Common branch workflow

We have several branches and want to develop a new feature. After developing on that branch, we want to add that code to the development branch.

1. `git branch` – See your branches

		$ git branch
		* master 			# production branch
		  development

2. `git checkout -b new-feature` – Check into new-feature branch

		$ git branch
		  master 			# production branch
		  development
		* new-feature
3. Develop on the new-feature branch and fix bugs on the new-feature
4. `git add -A` – Add all changes
5. `git commit -m "fixed bugs for new-feature"` – Commit changes
6. `git push origin new-feature` – Push the feature branch and commmits
7. `git checkout development` – Check into the development branch
8. `git merge new-feature` – Merge the new changes into the development branch

#### Dive Deeper

[http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/)

Look into resolving git merges, which can get hairy when many people are working on the same codebase.

Look into pull requests to start helping out open source projects.

Another, more extensive Git workflow: [https://github.com/Kunena/Kunena-Forum/wiki/Create-a-new-branch-with-git-and-manage-branches](https://github.com/Kunena/Kunena-Forum/wiki/Create-a-new-branch-with-git-and-manage-branches)