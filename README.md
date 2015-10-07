Androgogic - Alfresco Login Reset
---

This project includes both -repo and -share AMPs that override the default login page and adds the "missing" feature of the "reset password".
Also, the password is never shown in plain text. The entire process replicates in a similar way how Alfresco Cloud reset password works, through use of Activiti workflows, and using a unique instance key and ID to match with the original user request.

###This extension is being actively developed in [@Androgogic](https://twitter.com/androgogue) by their Alfresco Team. Credits are mentioned at the end of this file.

# Features

- Brand new Alfresco Share login page based on [Material Design by Google](http://www.google.com/design/spec/material-design/introduction.html).
- Front end framework based on a slightly modified version of [materialize.css](http://materializecss.com/). The GitHub project can be found [here](https://github.com/Dogfalo/materialize).
- A new Alfresco Share "Forgot Password" page that allows users to input their emails or username registered against the repository. The extension will then send an email with the instructions (even if multiple users with the same email are found).
- An Activiti workflow started for each valid request with a unique key ID.
- A new Alfresco Share "Reset Password" page that will match the `activiti$id` and the unique key ID with the user, and will prompt the list of available users (if multiple with the same email are found) in the reset password form. The user will then be able to chose from one of his accounts to reset the password for.
- A list of repository tier back end web scripts that manage and handle all requests, send emails and return status messages for Share ftl pages to be exposed.
- A scheduled action that ends all workflows that are part of unused password reset requests or older than 24 hours.

This process will allow a more solid and secure approach to reset password requests.
It replicates the current [Alfresco Cloud](https://my.alfresco.com/share/) behaviour.
The main improvement over other Alfresco reset password extensions is that no plain text passwords are sent through emails.
The user will be able to change it's own password as long as the `activiti$id`, the unique key ID and the email/username match.
The Share login page will also call the Alfresco `/api/server` web script and allow users to see if the repository is available, instead of trying to login and being prompted with an annoying message.

# Essentials

- Alfresco Enterprise/Community 5.x
- Alfresco Maven SDK 2.0.0 ([Alfresco Maven compatibility matrix](http://docs.alfresco.com/5.0/concepts/alfresco-sdk-compatibility.html))
- [Alfresco Maven Enterprise account](http://docs.alfresco.com/5.0/concepts/alfresco-sdk-tutorials-alfresco-enterprise.html) (if using Enterprise artifacts, `pom.xml` needs to be changed accordingly).
- `alfresco-login-reset-repo.amp` found [here](https://github.com/teqnology/alfresco-login-reset-repo)
- `alfresco-login-reset-share.amp` found [here](https://github.com/teqnology/alfresco-login-reset-share)

# Quickstart

- Locate the AMP files inside the path:
	- `alfresco-login-reset-repo/target/alfresco-login-reset-repo.amp` you can download it [here](https://github.com/teqnology/alfresco-login-reset-repo/blob/master/target/alfresco-login-reset-repo.amp)
	- `alfresco-login-reset-share/target/alfresco-login-reset-share.amp` you can download it [here](https://github.com/teqnology/alfresco-login-reset-share/blob/master/target/alfresco-login-reset-share.amp)
- Stop Alfresco
- Copy the [alfresco-login-reset-repo.amp](https://github.com/teqnology/alfresco-login-reset-repo) inside your alfresco `amps` folder
- Copy the [alfresco-login-reset-share.amp](https://github.com/teqnology/alfresco-login-reset-share) inside your alfresco `amps_share` folder
- Run `bin/apply_amps.sh` in order to install the extensions
- Start Alfresco
- Open Alfresco Share URL (eg. `http://localhost:8080/share`) to check if the new login page and the reset password are in place

# Quickstart for devs

## alfresco-login-reset-repo

- Enter the project folder and run `./run.sh` (you might need to `chmod u+x run.sh` to make it executable). This will setup the SDK and the Alfresco repository (not Share or Solr).
- Wait for the startup of the webapp and then go `http://localhost:8080/alfresco`.
- Import the project in your favorite IDE (with Maven integration) and start developing right away

For additional info please refer to [Maven Alfresco SDK documentation - Repository AMP Archetype](https://artifacts.alfresco.com/nexus/content/groups/public/alfresco-sdk-aggregator/latest/archetypes/alfresco-amp-archetype/index.html).

## alfresco-login-reset-share

- Enter the project folder and run `./run.sh` (you might need to `chmod u+x run.sh` to make it executable). This will setup the SDK and the Alfresco Share on port 8081 (not Alfresco Repository or Solr).
- Wait for the startup of the webapp and then go `http://localhost:8081/share`. NOTE: For this to work you need to have an Alfresco previously running on port 8080, for example as provided by the Alfresco AMP archetype
- Import the project in your favorite IDE (with Maven integration) and start developing right away.

For additional info please refer to [Maven Alfresco SDK documentation - Share AMP Archetype](https://artifacts.alfresco.com/nexus/content/groups/public/alfresco-sdk-aggregator/latest/archetypes/share-amp-archetype/index.html).

## (optional) working without alfresco-login-reset-repo

Often on local dev env there is no SMTP configuration. In order to test the project properly (the send email action will throw an error otherwise) you might want to configure the `alfresco-login-reset-share` project to connect to a working remote repository.
In case you want to make changes to the share-tier project only, you can update the share config file  `alfresco-login-reset-share/src/main/resources/META-INF/share-config-custom.xml`. Just locate the `config evaluator="string-compare" condition="Remote">` section and update it's `<endpoint-url>` values accordingly.

# Source code documentation

## Alfresco Explorer Web Script family `/androgogic`

### `forgot-password-workflow`

This web script is triggered when the user requests a reset password through the `forgot-password` Share page.
It checks on input provided, and sends an email (if any users is found) with a different message if the provided email is associated with one or multiple users.
The web script will then create a new workflow generating a unique key ID and a unique `activiti$id`.
The email will contain a link to update the password, along with the previously generated unique key ID and the `activiti$id`.
The link will be available for a one time use only. If not used, the link will expire in 24 hours automatically through a scheduled action.

- `POST /alfresco/service/androgogic/login/forgot-password`
- Creates a new workflow for the forgot password request
- xml configuration file (you can chose how the unique key ID is generated). This part has been ported but heavily modified from [share-extras/reset-password-dialog](https://github.com/share-extras/reset-password-dialog).

		<forgot-password>
			<key>16</key
			<chars>0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz</chars>
			<disallowed-users>admin</disallowed-users>
		</forgot-password>

- @method POST and @param `${email}` in JSON format. `${email}` accepts either standard email formatting (eg. example@hostname.com) or a simple username (eg. johndoe).

		/**
		 * Workflow forgot password script
		 * 
		 * @method POST
		 * @param
		 * {
		 *    email: ${email};
		 * }
		 * 
		 */  

### `reset-password-workflow`

- `POST /alfresco/service/androgogic/login/reset-password`
- Resets the password for the selected user if the specified workflow exists (pairs the `activiti$id` and it's unique key ID with provided URL arguments).
- @method POST and @param `${password}`. The confirmation password check is executed client side before the ajax call.

		/**
		 * Workflow reset password script
		 * 
		 * @method POST
		 * @param
		 * {
		 *    password: ${password};
		 * }
		 * 
		 */
- JSON result. The `${message}` is injected in the reset-password page notifying the user about the outcome of the password update.

		<#escape x as jsonUtils.encodeJSONString(x)>
		{
		   "success": ${result?string},
		   "message": "${message}"
		}
		</#escape>

### `list-users-workflow`

- `POST /alfresco/service/androgogic/login/list-users`
- Lists multiple users with the supplied email if the specified workflow exists
- @method POST and @param `${email}` in JSON format. `${email}` is used to find the list of users associated with that account and expose the list in the `reset-password` page

		/**
		 * Workflow reset password script
		 *    
		 * @method POST
		 * @param
		 * {
		 *     email: ${email};
		 * }
		 * 
		 * */

- JSON result. The array will contain either one entry (if a single users is returned) or an array of users associated with the same email. Note that `${u}` is not the user object itself, but just the username string.

		<#escape x as jsonUtils.encodeJSONString(x)>
		{
		   "success": ${result?string},
		   "message": "${message}",
		   "users": [
			   <#list users as u>
			   	{
		            "user": "${u}"
		        }<#if u_has_next>,</#if>
			   </#list>
		   ]
		}
		</#escape>

## ACP bootstrap

The HTML5 email templates are stored in the acp file inside the `/src/main/amp/config/alfresco/bootstrap` folder.
There are two ACPs to be deployed:

- `alfresco-email-template-bootstrap.acp` which contains:
	- `forgot-password-email.ftl`
	- `reset-password-email.ftl`
	- will be exploded and deployed inside the `/Data Dictionary/Email Templates/custom-email-template` folder during the first restart after the AMP deployment. The emails are used by the forgot-password and reset-password web scripts.

- `end-workflow-bootstrap.acp` which contains:
	- `end-expired-workflow-script.js`
		
			<import resource="classpath:alfresco/module/alfresco-login-reset-repo/reset-password-workflow.js">
			endExpiredWorkflow();

	- it will be exploded and deployed inside the `/Data Dictionary/Scripts/` folder during the first restart after the AMP deployment. It contains a single script that will be called by the scheduled action described below.

## Scheduled Action

The bean id `endResetPasswordWorkflow` declared in `/src/main/amp/config/alfresco/alfresco-login-reset-scheduled-action-services-context.xml` will be triggered every 10 minutes to check and close/end:
	- all the reset-password workflows that have been active for at least 24 hours
	- all the reset-password workflows that haven't been used in the last 24 hours
	- all the reset-password workflows that have been used and are currently in the completed state.

## Alfresco Custom Theme

The theme files are located in:

- `src/main/amp/config/alfresco/web-extension/site-data/themes` where the theme xml file is defined. The default one is called `customTheme.xml`. Use a similar one for a new theme deployment.
- `src/main/amp/web/themes/{nameAsXmlThemeFile}` (eg. `customTheme`) where all the theme's assets are stored. The structure reflects the original theme:
    - `/images` where theme images are stored.
    - `/yui/assets` where different ui components are stored
    - `/presentation.css` the main theme styling theme

### Share Login Theme

The login is no longer managed by the theme customization files. The Share pages are built through Surf platform Freemarker templates and JavaScript APIs.
If you want to update the login and reset password pages style you might want to look into the files stored in these paths:

- `alfresco-login-reset-share/src/main/amp/config/alfresco/templates/com/androgogic/base/login` where are stored share freemarker pages (login, forgot-password and reset-password)
- `alfresco-login-reset-share/src/main/amp/web/css/` where css and style files are stored. Remember that `main.css` overrides the `materialize.min.css` styling rules.
- `alfresco-login-reset-share/src/main/amp/web/js/` where script files are stored.

## Alfresco Share custom login extension

This is a completely new page, based on `materializecss` project (currently Alpha 0.96.1), with a `login.js` controller.
The controller calls a public web script:

	function getServer() {
	    var srv = remote.call("/api/server")
	    if (srv.status == 200) {
	      srvObj = eval("(" + srv + ")");
	      model.srv = srvObj;
	    }
	}
and injects `model.srv` data into the html login page.

This allows the Share login page to prompt users with a message if the Alfresco repository is down or unreachable.

In order to override the default login page, `share-config-custom.xml` needs to be updated as follows:

	<alfresco-config>
	   <config evaluator="string-compare" condition="WebFramework">
	      <web-framework>
	         <defaults>
	            <page-type>
	               <id>login</id>
	               <page-instance-id>login</page-instance-id>
	            </page-type>
	         </defaults>
	      </web-framework>
	   </config>
	</alfresco-config>

# Credits

- [Androgogic](http://www.androgogic.com/)
- [Alen Subat](http://alensubat.me/), blog: [curiousnerd.me](http://www.curiousnerd.me/)

> Written with [StackEdit](https://stackedit.io/).