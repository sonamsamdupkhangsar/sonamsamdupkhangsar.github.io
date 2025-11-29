# How-to page for SpringAuth app
This is the how-to page for this autorization app.

## Usage
A user can sign-up for an account and then create an OAuth2 client.  

If you need user roles then it needs to be configured using this SpringAuth app.
You can also add user using the SpringAuth app and associate them to your default organization.
The SpringAuth also provides ability to promote them to "SuperAdmin" capability so they can hav admin privileges to manage in the organization.

The following is a short summary of how to use this SpringAuth app.

## User Signup
A user can create an account using the [sign up](https://authorization.sonam.cloud/issuer/signup) or going from the authorization [home page](https://authorization.sonam.cloud/issuer/).

 ![saved client](images/signup/signup.png) 

Here I am signing up a user called `Tomato`. 

![tomat user signup](images/signup/tomatosoup-signup.png)

After clicking on signup you will see the following update on the signup page. 

![user signup success](images/signup/tomatosoup-user-created.png)


Check the email account that was used in the sign-up.  The user will get a email informing them to activate their account.

![activate account](images/signup/activateaccount.png).  

You have to activate the account within 2 hours or the user will have to request another email for activation by going to [login page](https://authorization.sonam.cloud/issuer/) and clicking on Email activation link.

![login page](images/signup/login.png)

### Account Activation

Once you click on the email activation link a confirmation text message will be shown. If the activation duration has expired then you will see a message saying the account activation window has expired.  You will have to request a new link using `Email activation link` when expired.

A user can only login and use this application on active accounts only.  The following shows the activation confirmation:
![account activated](images/signup/accountactivated.png)


## Sign-in

Go to [authzmanager](https://authzmanager.sonam.cloud/) and click on login link.  This will show the following sign-in page.

![sign-in page](images/signin.png)

Whn a user has signed-in with their username/password [https://authorization.sonam.cloud/issuer/signup] they will see the dashboard: ![dashboard](images/dashboard.png)

## How to use this app
You can use this app by just creating a [Oauth2 client](#create-oauth2-client) and integrating it with your app.  
If you need to assign user roles you can do that thru creating [Roles](#create-a-role) page and assigning them to Oauth2 client users.
You can also [Add user](#add-user) to organization.


## Create role
Click on the [Roles](https://authzmanager.sonam.cloud/admin/roles). 

![roles page](images/role/rolespage.png)

Click on `Create Role` link to add a role.

![enter role name](images/role/rolenameedit.png)

Once you submit the form the role is now created.

![role created](images/role/rolecreated.png)



## Create OAuth2 Client
Go to [Clients](https://authzmanager.sonam.cloud/admin/clients) page which will display the clients page:

![clients page](images/oauth2client/clients.png)

Click on "Create Client" to create a new OAuth2 Client and will show the following form page:

![new OAuth2 client](images/oauth2client/clients-new.png)

To create new OAuth2 client enter the following information:
1. Enter a client name for the client-id subpart.
2. Enter secret.
3. Select Client Authentication methods
4. Select Authorization Grant types
5. Select scopes needed for the OAuth2 client
6. Enter your client redirect uris.
7. Enter submit to create the client.


I have created a public pkce-client and this images shows it as an example:

![public OAuth2 client](images/oauth2client/pkce-client.png)    



## Add User
In this section we will create a user. Go to [Add User](https://authzmanager.sonam.cloud/admin/organizations/users?continue) link and see the following add user form:

![Add User](images/adduser/adduser.png)

Enter user's information here and choose whether to set their password for them and to activate their account as well:

![User info added for add user](images/adduser/adduser-form-filled.png)

After submitting the form the user will be created and see the following information:

![User added from add user](images/adduser/adduser-created.png)

The user is now added to the same organization as the one you created when you signed up or your default organization.  You can view the newly created user in the users tab:

![users in default org](images/oauth2client/users-in-org.png)

## Associate user and role
This part will associate the user we created to a role for a OAuth2 client app.

Go to [Clients](https://authzmanager.sonam.cloud/admin/clients) page. 

![clients page](images/oauth2client/clients.png)

Click on the pkce-client OAuth2 client and should see client details.

![public OAuth2 client](images/oauth2client/pkce-client.png)    


Click on the `Set User Role` tab link and will find the users in this org displayed that can be associated to this client.  You can also see the roles that can be assigned to the user.

![set user role](images/oauth2client/set-user-role.png)

I will assign the new user we created to admin role by clicking on submit on the user row and the update page reflects the role assignment.

![set user role assigned](images/oauth2client/adduser-role-assigned.png)

# Lets see the user role come in through a public Pkce app

I will use Chrome's guest profile to open a new tab.  For this I will use the Public Pkce client I have created in a Typescript app.

I will enter nextauth.sonam.cloud in the url and see the javascript app. 

![nextauth](images/nextauth/nextauth-pkce-app.png)

I am going to click the `Sign in` button and enter the username and password I created earlier for the user "Tashi".


![new user signin](images/new-sign-in.png)

After signing in I am now logged-in to the nextauth app as Tashi.

![nextauth-logged-in](images/nextauth/tashi-nextauth-logged-in.png) 


Click on the `API` link and you will find the user information and their access token.

![nextauth api link](images/nextauth/nextauth-api.png)

I will copy the jwt token shown in this page and do a base64 decode to show their content using `jwt.io` web service with some redaction.

![jwt base64 decoded](images/nextauth/accesstoken-base64-decoded.png)

The decoded payload shows the user role `admin` and other user and token information.

