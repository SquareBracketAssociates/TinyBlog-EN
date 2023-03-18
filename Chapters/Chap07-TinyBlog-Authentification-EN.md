## Authentication and SessionIn this chapter we will develop a traditional scenario: the user should login to access to the administration part of the application. He does it using a login and password.Figure *@ApplicationArchitectureAdminHeader@* shows the architecture that we will reach in this chapter.![Authentication flow.](figures/ApplicationArchitectureAdminHeader.pdf width=75&label=ApplicationArchitectureAdminHeader)Let us start to put in place a first version that allows one to navigate between the part of TinyBlog renderedby the component `TBPostsListComponent` and a first draft of the administration component as shown in Figure *@SimpleAdminLink@*. This illustrates how to invoke a component.In the  following we  will  build and  integrate a  component managing  the  login based  on modal  interaction.This will illustrate how we can elegantly map filed inputs to instance variables of a component.Finally we will show how the user information is stored into the current session.### A Simple Admin Component \(v1\)Let us define a really super simple administration component. This component inherits from the class  `TBScreenComponent` as mentioned in previous chapters and illustrated in Figure *@ApplicationArchitectureAdminHeader@*.```TBScreenComponent subclass: #TBAdminComponent
   instanceVariableNames: ''
   classVariableNames: ''
   package: 'TinyBlog-Components'```We define a first version of the rendering method to be able to test our approach.```TBAdminComponent >> renderContentOn: html
   super renderContentOn: html.
   html tbsContainer: [
      html heading: 'Blog Admin'.
      html horizontalRule ]```### Adding 'admin' ButtonWe add now a button in the header of the site  \(component `TBHeaderComponent`\) so that the user can access to the admin as shown in Figure *@SimpleAdminLink@*.To do so, we modify the existing components:  `TBHeaderComponent` \(header\) et `TBPostsListComponent` \(public part\).![Simple link to the admin part.](figures/SimpleAdminLink.png width=100&label=SimpleAdminLink)Let us add a button in the header:```TBHeaderComponent >> renderContentOn: html
		html tbsNavbar beDefault; with: [
			 html tbsContainer: [
				self renderBrandOn: html.
				self renderButtonsOn: html
		]]``````TBHeaderComponent >> renderButtonsOn: html
   self renderSimpleAdminButtonOn: html``````TBHeaderComponent >> renderSimpleAdminButtonOn: html
	html form: [
	   html tbsNavbarButton
		   tbsPullRight;
		   with: [
            html tbsGlyphIcon iconListAlt.
            html text: ' Admin View' ]]```When you refresh the web browser, the admin buttin is present but it does not have any effect \(See Figure *@withAdminView1@*\).We should define a callback on this button \(message `callback:`\) to replace the current component \(`TBPostsListComponent`\) by  the administration component \(`TBAdminComponent`\).![Header with an admin button.](figures/withAdminView1.png width=80&label=withAdminView1)### Header RevisionLet us revise the definition of `TBHeaderComponent` by adding a new instance variable named `component` to store and access to the current component \(either post list or admin component\). This will allow us  to access to the component from the header.```WAComponent subclass: #TBHeaderComponent
	instanceVariableNames: 'component'
	classVariableNames: ''
	package: 'TinyBlog-Components'``````TBHeaderComponent >> component: anObject
   component := anObject

TBHeaderComponent >> component
   ^ component```We add a new class method.```TBHeaderComponent class >> from: aComponent
   ^ self new
		component: aComponent;
		yourself```### Admin Button ActivationWe modify the component instantiation in `TBScreenComponent` method to pass the component which will be under the header.```TBScreenComponent >> createHeaderComponent
	^ TBHeaderComponent from: self```Note that the method  `createHeaderComponent` is defined in the superclass`TBScreenComponent` and it is applicable to all the subclasses.We can add now the callback on the button:```TBHeaderComponent >> renderSimpleAdminButtonOn: html
	html form: [
	html tbsNavbarButton
		tbsPullRight;
		callback: [ component goToAdministrationView ];
		with: [
				html tbsGlyphIcon iconListAlt.
				html text: ' Admin View' ]]```We just need to define the method `goToAdministrationView` on the component `TBPostsListComponent`:```TBPostsListComponent >> goToAdministrationView
	self call: TBAdminComponent new```Before clicking on the admin button, you should renew the current session by clicking on 'New Session': it will recreate the component `TBHeaderComponent`. You should get the situation presented in Figure *@withAdminCom@*. The 'Admin' button allows one to access the admin part v1. Pay attention  not to click twice  on the admin button because  we do not manage it  yet for the admin  part. We willreplace it by a Disconnect button.![Admin component under definition.](figures/WithAdminComp.png width=80&label=withAdminCom)### 'disconnect' Button AdditionWhen we display the admin part, we will replace the header component by a new one.This new header will display a disconnect button.Let us define this new header component:```TBHeaderComponent subclass: #TBAdminHeaderComponent
	instanceVariableNames: ''
	classVariableNames: ''
	package: 'TinyBlog-Components'``````TBAdminHeaderComponent >> renderButtonsOn: html
   html form: [ self renderDisconnectButtonOn: html ]```The `TBAdminComponent` component must use this header:```TBAdminComponent >> createHeaderComponent
   ^ TBAdminHeaderComponent from: self```Now we can  specialize our new admin header  to display a disconnect button.```TBAdminHeaderComponent >> renderDisconnectButtonOn: html
   html tbsNavbarButton
      tbsPullRight;
      callback: [ component goToPostListView ];
      with: [
         html text: 'Disconnect '.
         html tbsGlyphIcon iconLogout ]``````TBAdminComponent >> goToPostListView
	self answer```What is see is that the message `answer` gives back the control to the component that calls it. So we go back the post list.Reset the current session by clicking on 'New Session'. Then you can click on the 'Admin' button, you should see now the admin v1 display itself with a 'Disconnect' button. This button allows on to go back the public part as shown in Figure *@SimpleAdminLink@*.#### call:/answer: NotionWhen you study the previous code, you see that we use the`call:`/`answer:` mechanism of Seaside to navigate between the components `TBPostsListComponent` and `TBAdminComponent`.The message`call:` replaces the current component with the one passed in argument and gives it the flow of control.The message `answer:` returns a value to this call and gives back the flow of control to the calling argument.This mechanism is really poweful and elegant. It is explained in the vidéo 1 of week 5 of the Pharo Mooc \([http://rmod-pharo-mooc.lille.inria.fr/MOOC/WebPortal/co/content\_5.html](http://rmod-pharo-mooc.lille.inria.fr/MOOC/WebPortal/co/content_5.html)\).### Modal Window for AuthenticationLet us develop now a authentication component that when invoked will open a dialog box to requestthe login and password. The result we want to obtain is shown in Figure *@authentification@*.There are are some libraries of components ready to be used.For example, the Heimdal project available at [http://www.github.com/DuneSt/](http://www.github.com/DuneSt/) offers an authentication component or the Steam project [https://github.com/guillep/steam](https://github.com/guillep/steam) offers ways to interrogate google ou twitter accounts.![Authentication component.](figures/Authentification.png width=75&label=authentification)#### Authentication Component DefinitionWe define a new subclass of  `WAComponent` and its accessors.This component contains a login, a password and a component which invoked it to access to the admin part.```WAComponent subclass: #TBAuthenticationComponent
   instanceVariableNames: 'password account component'
   classVariableNames: ''
   package: 'TinyBlog-Components'``````TBAuthenticationComponent >> account
   ^ account``````TBAuthenticationComponent >> account: anObject
   account := anObject``````TBAuthenticationComponent >> password
   ^ password``````TBAuthenticationComponent >> password: anObject
   password := anObject``````TBAuthenticationComponent >> component
   ^ component``````TBAuthenticationComponent >> component: anObject
   component := anObject```The instance  variable `component`  will be initialized  by the following  class method:classe suivante :```TBAuthenticationComponent class >> from: aComponent
   ^ self new
      component: aComponent;
      yourself```### Authentication Component RenderingThe following method `renderContentOn:` defines the contents of a dialog box with the ID  `myAuthDialog`.This ID will be used to select the component that should be made visible when in modal mode.This dialog box has a header and a body. Note the use of the messages `tbsModal`, `tbsModalBody:`, and `tbsModalContent:`  which supports a modal interaction with the component.```TBAuthenticationComponent >> renderContentOn: html
   html tbsModal
      id: 'myAuthDialog';
      with: [
         html tbsModalDialog: [
            html tbsModalContent: [
               self renderHeaderOn: html.
               self renderBodyOn: html ] ] ]```The header displays a button to close the dialog box and a title with large fonts.Note that you can also use the ESC key to close the modal window box.```TBAuthenticationComponent >> renderHeaderOn: html
   html
      tbsModalHeader: [
         html tbsModalCloseIcon.
         html tbsModalTitle
            level: 4;
            with: 'Authentication' ]```The body of the component displays the input field for the login identifier, password and some buttons.```TBAuthenticationComponent >> renderBodyOn: html
    html
        tbsModalBody: [
            html tbsForm: [
                self renderAccountFieldOn: html.
                self renderPasswordFieldOn: html.
                html tbsModalFooter: [ self renderButtonsOn: html ] ] ]```The method `renderAccountFieldOn:` shows how the value of an input field is passed and stored in an instance variable of a component when the user finishes its input.The parameter of the `callback:` message is a bloc which takes as argument the value of the input field.```TBAuthenticationComponent >> renderAccountFieldOn: html
   html
      tbsFormGroup: [ html label with: 'Account'.
         html textInput
            tbsFormControl;
            attributeAt: 'autofocus' put: 'true';
            callback: [ :value | account := value ];
            value: account ]```The same process is used for the password.```TBAuthenticationComponent >> renderPasswordFieldOn: html
   html tbsFormGroup: [
      html label with: 'Password'.
      html passwordInput
         tbsFormControl;
         callback: [ :value | password := value ];
         value: password ]```Finally in the following `renderContentOn:` method, two buttons are added at the bottom of the modal window.The `'Cancel'` button which allows one to close the window using the attribute 'data-dismiss' and the `'SignIn'`button which sends the `validate` using a callback.The enter key is bound to the `'SignIn'` button activation when using the method `tbsSubmitButton`. This method sets the 'type' attribute to 'submit'.```TBAuthenticationComponent >> renderButtonsOn: html
   html tbsButton
      attributeAt: 'type' put: 'button';
      attributeAt: 'data-dismiss' put: 'modal';
      beDefault;
      value: 'Cancel'.
   html tbsSubmitButton
      bePrimary;
      callback: [ self validate ];
      value: 'SignIn'```In the `validate` method, we simply send a message to the main component giving it the information entered by the user.```TBAuthenticationComponent >> validate
	^ component tryConnectionWithLogin: self account andPassword: self password```%  !!!!! Améliorations%  Rechercher une autre méthode pour réaliser l'authentification de l'utilisateur (utilisation d'un backend de type base de données, LDAP ou fichier texte). En tout cas, ce n'est pas à la boite de login de faire ce travail, il faut le déléguer à un objet métier qui saura consulter le backend et authentifier l'utilisateur.%  De plus le composant ==TBAuthenticationComponent== pourrait afficher l'utilisateur lorsque celui-ci est logué.### Authentication Component IntegrationTo integrate our authentication component, we modify the Admin button of the headercomponent \(`TBHeaderComponent`\) as follows:```TBHeaderComponent >> renderButtonsOn: html
   self renderModalLoginButtonOn: html``````TBHeaderComponent >> renderModalLoginButtonOn: html
   html render: (TBAuthenticationComponent from: component).
   html tbsNavbarButton
      tbsPullRight;
      attributeAt: 'data-target' put: '#myAuthDialog';
      attributeAt: 'data-toggle' put: 'modal';
      with: [
         html tbsGlyphIcon iconLock.
         html text: ' Login' ]```The method `renderModalLoginButtonOn:` starts by rendering the component `TBAuthenticationComponent` within this web page.This component is created during each display and it does not have to be returned by the`children` method.In addition, we add 'Login' button with a icon lock.When the user clicks on this button, the modal dialog identified with the ID `myAuthDialog` will be displayed.Reloading the TinyBlog page, you should see now a 'Login' button in the header \(button that will pop up the authentication we just developed\) as illustrated by Figure *@authentification@*.### Naively Managing LoginsWhen you click on the 'SignIn' button you get an error.Using the Pharo debugger, you can see that we should define the method `tryConnectionWithLogin:andPassword:` on the component `TBPostsListComponent`since it is the one sent by the callback of the button.```TBPostsListComponent >> tryConnectionWithLogin: login andPassword: password
   (login = 'admin' and: [ password = 'topsecret' ])
         ifTrue: [ self goToAdministrationView ]
         ifFalse: [ self loginErrorOccurred ]```For the moment we store directly the login and password in the method and this is not really a good practice.### Managing ErrorsWe defined the method  `goToAdministrationView`. Let us add the method `loginErrorOccured` and a mechanismto display an error message when the user does not use the correct identifiers as shown in Figure *@loginErrorMessage@*.For this we will add a new instance variable `showLoginError` that represents the fact that we should display an error.```TBScreenComponent subclass: #TBPostsListComponent
   instanceVariableNames: 'currentCategory showLoginError'
   classVariableNames: ''
   package: 'TinyBlog-Components'```The method `loginErrorOccurred` specifies that an error should be displayed.```TBPostsListComponent >> loginErrorOccurred
	showLoginError := true```We add a method to test this state.```TBPostsListComponent >> hasLoginError
   ^ showLoginError ifNil: [ false ]```We define also an error message.```TBPostsListComponent >> loginErrorMessage
   ^ 'Incorrect login and/or password'```We modify the method `renderPostColumnOn:` to perform a specific task to handle the errors.```TBPostsListComponent >> renderPostColumnOn: html
   html tbsColumn
      extraSmallSize: 12;
      smallSize: 10;
      mediumSize: 8;
      with: [
         self renderLoginErrorMessageIfAnyOn: html.
         self basicRenderPostsOn: html ]```The method `renderLoginErrorMessageIfAnyOn:` displays if necessary an error message.It sets the instance variable `showLoginError` so that we do not display the errorundefinitely.```TBPostsListComponent >> renderLoginErrorMessageIfAnyOn: html
   self hasLoginError ifTrue: [
      showLoginError := false.
      html tbsAlert
         beDanger ;
         with: self loginErrorMessage
   ]```![Error message in case wrong identifiers.](figures/LoginErrorMessage.png width=75&label=loginErrorMessage)### Modeling the AdminWe do not want to store the administrator identifiers in the code as we did previously. We revise this now and will store the identifiers in a model: a class `Admin`.Let us start to enrich our TinyBlog model with the notion of administrator. We define a classnamed `TBAdministrator` characterized by it pseudo, login and password.```Object subclass: #TBAdministrator
	instanceVariableNames: 'login password'
	classVariableNames: ''
	package: 'TinyBlog'``````TBAdministrator >> login
   ^ login``````TBAdministrator >> login: anObject
   login := anObject``````TBAdministrator >> password
   ^ password```Note that we do not store the admin password in the instance variable `password` but its hash encoded in SHA256.```TBAdministrator >> password: anObject
   password := SHA256 hashMessage: anObject```We define also a new instance creation method.```TBAdministrator class >> login: login password: password
	^ self new
			login: login;
			password: password;
			yourself```You can verify that the model works by executing the following expression:```luc := TBAdministrator login: 'luc' password: 'topsecret'.```### Blog adminWe decide for simplicity that a blog has one admin.We add the instance variable `adminUser` and an accessor in the classe `TBBlog` to storethe blog admin.```Object subclass: #TBBlog
	instanceVariableNames: 'adminUser posts'
	classVariableNames: ''
	package: 'TinyBlog'``````TBBlog >> administrator
   ^ adminUser```We define a default login and password that we use as default.As we will see later, we will modify such attributes and these modified attributes will besaved at the same time that the blog in a database.```TBBlog class >> defaultAdminPassword
   ^ 'topsecret'``````TBBlog class >> defaultAdminLogin
   ^ 'admin'```Now we create a default admin.```TBBlog >> createAdministrator
	^ TBAdministrator
			login: self class defaultAdminLogin
			password: self class defaultAdminPassword```And we initialize the blog to set a default administrateur.```TBBlog >> initialize
   super initialize.
   posts := OrderedCollection new.
   adminUser := self createAdministrator```### Setting a New AdminWe should not recreate the blog:```	TBBlog reset; createDemoPosts```We can now modify the admin information as follows:```|admin|
admin := TBBlog current administrator.
admin login: 'luke'.
admin password: 'thebrightside'.
TBBlog current save```Note that without doing anything, the blog admin information has been saved by Voyage in the database.Indeed the class `TBBlog` is a Voyage root, all its atttributes are automatically stored in the database when it receivedthe message `save`.#### Possible EnhancementsDefine some tests for the extensions by writing new unit tests.### Integrating the Admin InformationLet us modify the method `tryConnectionWithLogin:andPassword:` so that it uses the current blog admin identifiers.Note that we are comparing the hash SHA256 of the password since we do not store the password.```TBPostsListComponent >> tryConnectionWithLogin: login andPassword: password
   (login = self blog administrator login and: [
      (SHA256 hashMessage: password) = self blog administrator password ])
         ifTrue: [ self goToAdministrationView ]
         ifFalse: [ self loginErrorOccurred ]```### Storing the Admin in the Current SessionWith the current setup, when the blog admin wants to navigate between the private and public part, he must reconnectseach time.We will simplify this situation but storing the current admin information in the session when the connection issuccesful.A session object is given to the each instance of the application.Such session allows on to keep information which are shared and accessible between components.We will then store the current admin in a session and modify the components to display buttons that supporta simplified navigation when the admin is logged.When he explicitely disconnect or when the session expires, we delete the current session.Figure *@SessionNavigation@* shows the navigation between the pages of TinyBlog.![Navigation and identification in TinyBlog.](figures/sessionAuthSimplifiedNavigation.pdf width=100&label=SessionNavigation)### Definition and use of specific sessionLet us start to define a subclass of `WASession` and name it `TBSession`. We add in this new class an instance variable that stores the current admin.```WASession subclass: #TBSession
    instanceVariableNames: 'currentAdmin'
    classVariableNames: ''
    package: 'TinyBlog-Components'``````TBSession >> currentAdmin
    ^ currentAdmin``````TBSession >> currentAdmin: anObject
    currentAdmin := anObject```We define a method `isLogged` allows one to know if the administration is logged.```TBSession >> isLogged
    ^ self currentAdmin notNil```Now we should indicate to Seaside to use  `TBSession` as the class of the current session for our application.This initialization is done in the class method `initialize` in the class `TBApplicationRootComponent` as follows:```TBApplicationRootComponent class >> initialize
      "self initialize"
      | app |
      app := WAAdmin register: self asApplicationAt: 'TinyBlog'.
      app
         preferenceAt: #sessionClass put: TBSession.
      app
         addLibrary: JQDeploymentLibrary;
         addLibrary: JQUiDeploymentLibrary;
         addLibrary: TBSDeploymentLibrary```Do not forget to exectute this expression `TBApplicationRootComponent initialize` before testing the application.### Storing the Current AdminWhen a connection is successful, we add the admin object to the current session using the message `currentAdmin:`.Note that the current session is available to every Seaside component via `self session`.```TBPostsListComponent >> tryConnectionWithLogin: login andPassword: password
   (login = self blog administrator login and: [
      (SHA256 hashMessage: password) = self blog administrator password ])
         ifTrue: [
            self session currentAdmin: self blog administrator.
            self goToAdministrationView ]
         ifFalse: [ self loginErrorOccurred ]```### Simplified navigationTo put in place the simplified navigation we discussed above, we modify the header to display either a login button or aa simple navigation button to the admin part without forcing any reconnection. For this we use the session and the fact that we can know if a user is logged.```TBHeaderComponent >> renderButtonsOn: html
    self session isLogged
        ifTrue: [ self renderSimpleAdminButtonOn: html ]
		  ifFalse: [ self renderModalLoginButtonOn: html ]```You can test this new navigation but first create a new session  \('New Session' button\).One reconnected the admin is added in session.Note that the deconnection button does not work correctly since it does invalidate the session.### Managing DeconnectionWe add a method  `reset` on our session object to delete the current admin, invalidate the current session and redirect to the application entry point.```TBSession >> reset
   currentAdmin := nil.
	self requestContext redirectTo: self application url.
	self unregister.```Now we modify the header deconnection button to send the message `reset` to the correct session.```TBAdminHeaderComponent >> renderDisconnectButtonOn: html
   html tbsNavbarButton
      tbsPullRight;
      callback: [ self session reset ];
      with: [
         html text: 'Disconnect '.
         html tbsGlyphIcon iconLogout ]```Now we  'Disconnect' button works the way it should.### Simplified Navigation to the Public PartWe can add now a button in the header of the admin part to go back to the public part without being forced to get disconnected.```TBAdminHeaderComponent >> renderButtonsOn: html
	html form: [
		self renderDisconnectButtonOn: html.
		self renderPublicViewButtonOn: html ]``````TBAdminHeaderComponent >> renderPublicViewButtonOn: html
   self session isLogged ifTrue: [
      html tbsNavbarButton
         tbsPullRight;
         callback: [ component goToPostListView ];
         with: [
            html tbsGlyphIcon iconEyeOpen.
            html text: ' Public View' ]]```Now you can test the naviagtion. It should correspond to the situation depicted by Figure *@SessionNavigation@*.### ConclusionWe put in place an authentication for TinyBlog. We create a reusable modal component. We made the distinction betweencomponent displayed when a user is connected or ot not and ease the navigation of a connected user using session.We are now ready for the administration part of the application and we will work on this in the next chapter.We will take advantage of it to show and advanced aspect: the automatic form generation.#### Possible EnhancementsYou can:- Add the admin logging in the header- Manage multipel admin accounts.