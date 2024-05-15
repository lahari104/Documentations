### Creating a GitLab account:
------------------------------

* First we need to create a GitLab account using the official mail-ID of Insignia (<user-name>@insigniacs.us)

[GitLab SignUp](https://gitlab.com/users/sign_up)

* Give the required details(UserName, Mail and Password) and register with GitLab to create the account.

* Once the signup is done login to the GitLab with the credentails which created above

[Login GitLab](https://gitlab.com/users/sign_in)

![Preview](login page pic)

* After login to the GitLab dashboard we can create a project 

![Preview](create project pic)

* After creating the project we need to clone the project using `git clone <project-url>`.

* While cloning the project it will promt for credentials there we need to mention our mail and token.

* TO generate token go to the GitLab dashboard and open the project which we have created and select settings, in this we can see an option `access tokens` select this option and create a token.

* Use this token to authenticate with gitlab from local.

* Then make the changes in the project and then push the changes to project using below commands.

```bash
git add .
git commit -m <commit message>
git push -u origin <branch-name>
```