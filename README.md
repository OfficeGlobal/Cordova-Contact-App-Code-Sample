## Cordova Contact Client App

This sample app shows step by step how to create a Cordova app using Ionic framework and O365 Outlook services from scratch.

It is highly recommended to go through [Visual Studio Tools for Apache Cordova](http://www.visualstudio.com/en-us/explore/cordova-vs.aspx) and [Getting Started with Visual Studio Tools for Apache Cordova](http://msdn.microsoft.com/en-us/library/dn771545.aspx) to setup Cordova development environment.

In this tutorial, you'll these steps

1. Create Blank Codrova using Visual Studio
2. Add Ionic framework
3. Add O365 services to app
4. Set permissions to O365 contact tenet to grant appropiate access to app
5. Create app folder structure, UI routing and layout using Ionic controls and navigation
6. Acquire an access token and get the Outlook services client using AngularJS factory
7. Use O365 API to fetch contacts
8. Use O365 API to add new contact
9. Run the app!

![Application login page](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/login.png)
![Application contact view](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/contact-list.png)
![Application contact detail page](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/New-contact.png)

### Step 1: Create Blank Codrova using Visual Studio
Create a new Cordova project in Visual Studio by choosing File --> New project --> JavaScript --> Apache Cordova Apps --> Blank App template. This sample uses JavaScript code, but you can also write your Cordova app in TypeScript.

### Step 2: Add Ionic framework
1.	From the Ionic framework website, choose Download beta.
2.	Extract the zip
3.	Create new folder named lib under Cordova project in Visual Studio solution explorer and copy the extracted content under lib folder.

![Folder structure for project](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/Ionic.png)

**Update the script references**
- In index.html, add the following Ionic references in the ``` <head> ``` element, after the Cordova and platformOverrides script references.

```html
<script src="lib/ionic/js/ionic.bundle.min.js"></script>
```
- In index.html, add following ionic css reference.
```html
 <link href="lib/ionic/css/ionic.min.css" rel="stylesheet" />
```
### Step 3: Add O365 services to app
Refer [Set up your Office 365 development environment](http://msdn.microsoft.com/en-us/office/office365/howto/setup-development-environment) documentation on Signing up for an Office 365 Developer Site and Set up Azure Active Directory access for your Developer Site.

Follow these steps to add and configure Office 365 APIs by using the Services Manager in Visual Studio.

1. Download and install the Office 365 API tools from the Visual Studio Gallery
2. On the project node, right click and choose **Add --> Connected Service**
3. At the top of the Services Manager dialog box, choose the Office 365 link, and then choose Register your app. Sign in with a tenant administrator account for your Office 365 developer organization.
![O365 Services Manager dialog sign in](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/ServiceManager.png)

### Step 4: Set permissions to O365 contact tenet to grant appropiate access to app
Select Contact and click on Permissions... link on right pane and then select 'have full access to users' contact'. Similarly if you want to give only read access to app, select 'Read users' contact'.

![O365 permission scopes for contacts dialog](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/Contact-permission.png)

Click Apply and Ok to set the permission and add O365 API to project. This will add Service folder containing JavaScript libraries to the project.

![project folder tree after adding permissions](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/service-folder.png)

In index.html, add the following O365 references in the ``` <head> ``` element.
```html
<script src="services/office365/scripts/o365loader.js"></script>  
<script src="services/office365/settings/settings.js"></script>
```
**Step 5: Create app folder structure, UI routing and layout using Ionic controls and navigation**

1. Create app folder under project root node. app folder will contain files specific to app. Each UI component which does fetching and binding the data to UI will have corresponding controller much like UI and code behind pattern. For example contact-list.html will show list control to display user contacts and contact-list-ctrl.js will contain code to fetch users contact using O365 API.

![project folder tree structure for application](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/Contact-app-folders.PNG)

**Folder and file detail:**
- **auth** contains UI and code for signing-in and sign-out
- **app.js** contains ui routing to navigate to different pages
- **service-o365.js** contains utility function to get access token, create Outlook services client object, signout and get user name. This is implemented as Angular factory so that these functions can be exposed as utility function across different pages.

**app.js defining ui routing**
```javascript
angular.module("app365", ["ionic"])

.run(function ($ionicPlatform) {
    $ionicPlatform.ready(function () {
        if (window.StatusBar) {
            StatusBar.styleDefault();
        }
    });
})

.config(function ($stateProvider, $urlRouterProvider) {

    $stateProvider

    // Layout page
    .state('app', {
        abstract: true,
        url: "/app",
        templateUrl: "app/layout/layout.html"
    })

    // Sign-in page
     .state('sign-in', {
         url: "/sign-in",
         templateUrl: "app/auth/sign-in.html"
     })

    // Sign-out page
    .state('app.sign-out', {
        url: "/sign-out",
        views: {
            'mainContent': {
                templateUrl: "app/auth/sign-out.html"
            }
        }
    })    

    // Add new contact page
    .state('app.newContact', {
        url: "/newContact",
        views: {
            'mainContent': {
                templateUrl: "app/contact/add-contact.html"
            }
        }
    })

    // Contact list page
    .state('app.contact', {
        url: "/contact",
        views: {
            'mainContent': {
                templateUrl: "app/contact/contact-list.html"
            }
        }
    });    

    // Navigate to sign-in page when app starts.
    $urlRouterProvider.otherwise('sign-in');
})
```

**App layout (menu, nav-bar)**
```html
<ion-side-menus ng-controller="layoutCtrl as vm">
    <ion-pane ion-side-menu-content>
        <ion-nav-bar class="bar-positive">
            <ion-nav-back-button class="button-clear icon ion-ios7-arrow-back"></ion-nav-back-button>
            <button menu-toggle="left" class="button button-icon icon ion-navicon"></button>
        </ion-nav-bar>
        <ion-nav-view name="mainContent" animation="slide-left-right"></ion-nav-view>
    </ion-pane>

    <ion-side-menu side="left">
        <header class="bar bar-header bar-positive">
            <h1 class="title">{{vm.userName}}</h1>
        </header>
        <ion-content class="has-header">
            <ion-list>
                <ion-item nav-clear menu-close ui-sref="app.contact">Home</ion-item>
                <ion-item nav-clear menu-close ui-sref="app.newContact">New Contact</ion-item>                           
                <ion-item nav-clear menu-close ui-sref="app.sign-out">Sign-out</ion-item>
            </ion-list>
    </ion-side-menu>
</ion-side-menus>
```

### Step 6: Acquire an access token and get the Outlook services client using AngularJS factory

**Acquire an access token**
```javascript
var authContext = new O365Auth.Context();
authContext.getIdToken("https://outlook.office365.com/")
.then((function (token) {
     // Get auth token
     authtoken = token;
     // Get user name from token object.
     userName = token.givenName + " " + token.familyName;
    }), function (error) {
      // Log sign-in error message.
      console.log('Failed to login. Error = ' + error.message);
 });
```
**create Outlook services client object**
```javascript
var outlookClient = new Microsoft.OutlookServices.Client('https://outlook.office365.com/api/v1.0', authtoken.getAccessTokenFn('https://outlook.office365.com'));
```
### Step 7: Use O365 API to fetch contacts

**Fetch all contacts**
```javascript
outlookClient.me.contacts.getContacts().fetch()
.then(function (contacts) {
  // Get the current page. Use getNextPage() to fetch next set of contacts.
  vm.contacts = contacts.currentPage;
  $scope.$apply();                
});
```

### Step 8: Use O365 API to add new contact
Outlook client object can be used to add, update and delete contact.

**Create the page to submit the data for creating new contact**
```html
<ion-view title=" New Contact" ng-controller="newContactCtrl as vm">
    <ion-content class="has-header">
        <div class="list">
            <label class="item item-input">
                <input type="text" placeholder="First Name" ng-model="newContact.firstname" />
            </label> 
            <label class="item item-input">
                <input type="text" placeholder="Last Name" ng-model="newContact.lastname" />
            </label> 
            <label class="item item-input">
                <input type="email" placeholder="Email" ng-model="newContact.email">
            </label>
            <label class="item item-input">
                <input type="text" placeholder="Phone" ng-model="newContact.phone" />
            </label>            
        </div>       
        <div class="padding">
            <button class="button button-block button-positive" ng-click="addContact()">
                Add Contact
            </button>
        </div>
    </ion-content>
</ion-view>
```
**Use following code to create new contact**
```javascript 
// Outlook client object.
var outlookClient;
// To store contact info.
$scope.newContact = {};

$scope.addContact = function () {
   outlookClient = app365api.outlookClientObj();           

    // Contact object
    var contact = new Microsoft.OutlookServices.Contact();

    // First and last name
    contact.givenName = $scope.newContact.firstname;
    contact.surname = $scope.newContact.lastname;

    // Mobile phone
    contact.mobilePhone1 = $scope.newContact.phone;

    // Email address
    var emailAddress = new Microsoft.OutlookServices.EmailAddress();
    emailAddress.address = $scope.newContact.email;
    contact.emailAddresses.push(emailAddress);

    // Add Contact
    outlookClient.me.contacts.addContact(contact)
    .then((function (response) {
         console.log('Contact added successfully.');    
     })
     .bind(this), function (reason) {
         // Log the error message when add contact fails.
         console.log('Fail to add contact. Error = ' + reason.message);
      });
};
```

### Step 9: Run the app

1. Select Android and target either as Android Emulator or device. Please note currently Ripple is not supported for O365 auth.

![target platform selection bar](https://github.com/abhikum/mobiledev/blob/gh-pages/O365AppImages/Android%20-%20Run.PNG)

**F5 to run the app**

Refer [Deploy and Run Your App Built with Visual Studio Tools for Apache Cordova](http://msdn.microsoft.com/en-us/library/dn757049.aspx) on more detail on running the Cordova app across different platforms.
