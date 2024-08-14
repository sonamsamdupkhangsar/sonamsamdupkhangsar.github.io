# Create OAuth2 app
Once you are logged-in you can create a OAuth2 app by navigating to [Clients](https://authzmanager.sonam.cloud/admin/clients) page![clients](images/clients.png). 

In this example, I am going to create a OAuth2 app for a Java application.  This server side app will display the user's access-token when authenticated.  This is for demo purposes only.

Click on `Create Client` link.  I have pasted an image of my client ![oidc-private-client](images/client-create.png).


The `Client-id` field will be set with a UUID of type 4 in the text field.  It is auto-generated and appended with the `Enter client-id subpart` input.  The UUID helps to ensure that the client-id will be unique.

I have entered the `Client issued at` field using a calendar.  The secret is set to `hello`.  The other fields I have set are the Client Authentication Methods to be `CLIENT_SECRET_BASIC` for basic authentication for this client.  For Authorization grant types I have selected using Authorization Code.  I have also added some scopes that are available such as `OPENID`, `PROFILE` and a few custom ones like `read` and `write`.

I also provide the redirect uri once the user is authenticated to go back to the application url.

When you hit the submit button it will create the OAuth2 client and should see the saved client as follows ![saved client](images/client-saved.png)

# Create Organization
This Organization (org) is to group a OAuth2 client app to a org and to a Role.
I created one called `Yak Yak Yak` org using the [Organizations](https://authzmanager.sonam.cloud/admin/organizations)![yak yak yak](images/yak-org.png).

To assign roles to this organization, we need to create the roles by clicking on the `Roles` link ![Roles](images/roles.png).  

Create a simple role like `Admin` and `User` like shown![roles](images/roles-created.png).  When you create the roles you can also assign them to the organization ![assigned to Yak Yak Yak](images/admin_yakorg.png). I have assigned both roles to `Yak Yak Yak` organization.

Now if you go back to [Organizations](https://authzmanager.sonam.cloud/admin/organizations) you will see the roles assigned to it ![yak roles](images/yak_roles.png).

If you click on the `User association` tab you will see by default the person that created the org. You can also add other people by searching in the textfield by their username ![org user association](images/org-user-association.png).



Now we will go back to the Clients again to find the one we created earlier ![client list](images/client-list.png) and select that client.  However, this time we want to associate the client to the organization we created first ![client-org-association](images/client-org-association.png).


We then move to assign the user that is associated to the organization to be assigned a role in this OAuth2 Client ![user role assigned](images/org-user-assigned-role-oauth2-client.png).  You can select the Role for this user by selecting the available roles and assign them one ![selected](images/org-user-assigned-role-oauth2-client-done.png).

In this page, I created a organization, and a couple of roles and assigned them to the OAuth2 client.  In the next section, a Java app will use the client and get the roles. 