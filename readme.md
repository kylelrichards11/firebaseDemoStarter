# Kyle Richards Project 3 Part 2 Tutorial - Firebase

In this tutorial, we will be adding Firebase functionality to our existing "Pet War" app. You will need network connection throughout this tutorial.

# Start a Firebase Project

To use Firebase, you first have to register your project with Firebase at https://console.firebase.google.com/u/0/

Click on add project, give it a name, and then click create. I named my project PetWar.

Once the project has been created, you will see the project overview page. Click on the android icon above where it says **Add an app to get started**.

![Add an App](https://raw.githubusercontent.com/kylelrichards11/fileHosting/master/CS4518_Proj3_Part2_addApp.png)

Then, follow the instructions to link your firebase project to your app. 

1. Add your android package name, which can be found at the top of the MainActivity.java file
2. Download the config file and move to your app folder in the project view
3. Update your gradle files and sync gradle
4. Run your app and check to see if Firebase connected to it.

![Register App](https://raw.githubusercontent.com/kylelrichards11/fileHosting/master/CS4518_Proj3_Part2_registerApp.png)

You are now ready to start adding features to your Firebase Project!

# Firebase Authentication

Firebase provides multiple methods to authenticate users. We will be using a simple email and password signup. 

On the left hand side menu, click on **Authentication** and then click **Set up sign in method**.

For the Sign-in providers page, click on Email/Password. Enable the top option, and then click Save.

![Enable Authentication](https://raw.githubusercontent.com/kylelrichards11/fileHosting/master/CS4518_Proj3_Part2_enableAuth.png)

In your app's build.gradle file, add the following dependency.

`implementation 'com.google.firebase:firebase-auth:16.0.5'`

## Set up Login and Sign Up Screens

In your project, create two new empty activities, LoginActivity and SignUpActivity.

### Login Activity

Create a simple form that has two text fields and two buttons. The text fields will handle the user inputting their email and password. Have one button be a login button, and the other button be a sign up button. Finally, add some text that will display if the user failed to login.

We want this login screen to be the first screen users see when they open our app, so we need to specify this in our AndroidManifest.xml file. Move the following lines out of the `<activity>` tag for MainActivity and add it to the `<activity>` tag for LoginActivity.

```xml
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
```

### Sign Up Activity

Create a similar form for signing up, again with two text fields for email and password. This time, only include one button that says "Sign Up". To handle going back to the login page, we will define this activity's parent to be the LoginActivity in the Android Manifest. Add the following line to the activity tag in AndroidManifest.xml.

```xml
android:parentActivityName=".LoginActivity"
```

If you run your app now, you should be able to navigate back and forth between the Login and Sign Up screens. Next, we have to handle actually signing up and logging in.


## Connect Sign Up to Firebase

First, we need to instantiate a FirebaseAuth object. Define a variable called mAuth, and then add this line to your onCreate() function.

```java
mAuth = FirebaseAuth.getInstance();
```

Firebase provides a very straightforward way for signing users up with an email and password. Insert the following function in your SignUpActivity.

```java
private void registerNewUser(String email, String password) {
    mAuth.createUserWithEmailAndPassword(email, password).addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {

        @Override
        public void onComplete(@NonNull Task<AuthResult> task) {
            if (task.isSuccessful()) {
                // TODO: Handle successful sign up
            }
            else {
                // TODO: Handle unsuccessful sign up
            }
        }
    });
}
```

You can handle the unsuccessful sign up however you like. At least let the user know something is wrong, but Firebase also provides ways to tell the user what happened.

Now we need to get the email and password that the user input into our form and pass it into that function.

```java
public void onClickSignUpButton(View v) {
    EditText email = (EditText)findViewById(R.id.inputEmailSU);
    EditText password = (EditText)findViewById(R.id.inputPasswordSU);

    String emailString = email.getText().toString();
    String passwordString = password.getText().toString();

    // Make sure the input strings are not empty
    if (!emailString.isEmpty() && !passwordString.isEmpty()) {
        registerNewUser(emailString, passwordString);
    }
    else {
        TextView t = (TextView)findViewById(R.id.signUpError);
        t.setText("Must fill in email and password fields");
    }
}
```

Make sure this function is linked to the submit button in your sign up layout.

Finally, we need to navigate to our main activity once the user has successfully signed up. Insert the following function, and then call it in `registerNewUser()` when there is a successful sign up.

```java
private void goToMainActivity() {
    Intent intent = new Intent(this, MainActivity.class);
    startActivity(intent);
}
```

Try and sign up to your app! If all is successful, you should be redirected to the main screen. Additionally, you can check to see if your information has been added to Firebase. In the Firebase Console, click the Authentication tab on the left. Then in the middle-ish of the screen is a tab that says Users. Clicking on that will show a list of users, and you should see the email you just entered!

## Connect Login to Firebase

Now that users can sign up, they need to be able to login to your app. The process will be very similar to allowing users to sign up. In LoginActivity, instantiate a FirebaseAuth mAuth by calling `FirebaseAuth.getInstance()` in the onCreate() callback. We now need to check if our user is already logged in, since we do not want them to log in unnecessarily. Add the following override of onStart().

```java
@Override
public void onStart() {
    super.onStart();

    FirebaseUser currentUser = mAuth.getCurrentUser();

    // If we are already logged in, there is no need to login again.
    if(currentUser != null) {
        goToMainActivity();
    }
}
```

Then we need to create a function that allows users to log in.

```java
private void LogInUser(String email, String password) {
    mAuth.signInWithEmailAndPassword(email, password).addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
        @Override
        public void onComplete(@NonNull Task<AuthResult> task) {
            if (task.isSuccessful()) {
                goToMainActivity();
            } else {
                // TODO: Handle login error
            }
        }
    });
}
```

Now we need to get the email and password that the user input into our form and pass it into that function.

```java
public void onClickLoginButton(View v) {
    EditText email = (EditText)findViewById(R.id.inputEmail);
    EditText password = (EditText)findViewById(R.id.inputPassword);

    // Make sure the input strings are not empty
    String emailString = email.getText().toString();
    String passwordString = password.getText().toString();

    if (!emailString.isEmpty() && !passwordString.isEmpty()) {
        LogInUser(emailString, passwordString);
    }
    else {
        TextView t = (TextView)findViewById(R.id.loginError);
        t.setText("Must fill in email and password fields");
    }
}
```

Make sure this function is linked to the login button in your login layout.

## Sign Out

The last step in implementing authentication is allowing users to log out, which is very simple. We need to add a button to the user interface that allows users to sign out. Insert the following function to be completed when this Sign Out button is clicked.

```java
public void onClickSignOutButton(View v) {
    FirebaseAuth.getInstance().signOut();

    // After signing out, we want to go back to the login screen
    Intent intent = new Intent(this, LoginActivity.class);
    startActivity(intent);
}
```

# Firebase Realtime Database

In this section we will be using firebase as cloud storage for data. We want to store the number of people that think each cat is cute or not. Therefore, each time a user votes for a cat, we want to save their vote in the database. For simplicity, we will allow users to vote multiple times.

![Pet War](https://raw.githubusercontent.com/kylelrichards11/fileHosting/master/CS4518_Proj3_Part2_CatUI.png)

For each kitten, we want to keep track of how many users answered yes and how many users answered no.

## Gradle Setup

The first step is to add the following implementation to the app's build.gradle

`implementation 'com.google.firebase:firebase-database:16.0.5'`

NOTE: I also had to update the following two dependencies to avoid gradle errors

```
implementation 'com.google.android.gms:play-services-location:16.0.0'
implementation 'com.google.firebase:firebase-core:16.0.5'
```

## Enable Realtime Database

In the Firebase Console, click on **Database** on the left hand side. Then scroll down to Realtime Database and click Create Database. For now, set the database rules to be in test mode. This will allow you to skip the Authentication step above.

![Enable Database](https://raw.githubusercontent.com/kylelrichards11/fileHosting/master/CS4518_proj3_part2_DatabaseInit.png)

## Main Activity Set Up

First, we need to get a reference to our database. Define the following variable in your MainActivity.

```java
private FirebaseDatabase database;
```

Then in onCreate() initialize it by calling 

```java
database = FirebaseDatabase.getInstance();
```

## Update Database

NOTE: The function `changeQuestionText()` is a function I have to update a TextView in my UI. It can be replaced with any method to display text.

The first time any user runs our app, there will be no data for them to read. Therefore, we need to check if there is data before we can update it. The following code is an outline for how to do so.

```java
DatabaseReference rootRef = FirebaseDatabase.getInstance().getReference();
rootRef.addListenerForSingleValueEvent(new ValueEventListener() {
    @Override
    public void onDataChange(DataSnapshot snapshot) {
        final DatabaseReference ref = database.getReference("path");

        // If data to this path exists
        if (snapshot.hasChild("path")) {
            // Do something
        }
        else {
            // If the path did not exist, then we can write a new path instead of updating
        }
    }

    @Override
    public void onCancelled(DatabaseError error) {}
});
```

You'll notice we need to know the path to the data that we are potentially updating. Firebase's database is essentially a JSON object, so the path is a series of keys to get to the value you want. In our case, we are going to have the database structured such that each file name for the picture is a unique key that has a yes key and a no key. The yes and no keys will have values that represent how many votes each answer got as the answer to whether or not the cat in the picture is cute.

Therefore, we want to make the above code into a function and then pass the desired path into it. 

```java
private void updateAndWatchDB(final String path) {
    DatabaseReference rootRef = FirebaseDatabase.getInstance().getReference();
    rootRef.addListenerForSingleValueEvent(new ValueEventListener() {
        @Override
        public void onDataChange(DataSnapshot snapshot) {

            // Make sure that the path exists, if not, create it and set the value to 1
            if (snapshot.hasChild(path)) {
                // Do something
            }
            else {
                ref.setValue(1);  // If the path did not exist, then this is the first vote
            }
        }

        @Override
        public void onCancelled(DatabaseError error) {
            changeQuestionText("Could not read database");
        }
    });
}
```

We will need a variable to keep track of the name of the file that is currently displayed. Then, we will call this function in the yes and no button click handlers as follows

```java
public void onClickYesButton(View v) {
    updateAndWatchDB(currentPictureName+"/yes");
}

public void onClickNoButton(View v) {
    updateAndWatchDB(currentPictureName+"/no");
}
```

In the `updateAndWatchDB()` function, we handle the case when the path does not exist, by simply setting the value at that path to 1, which will create the path automatically for us. Now we have to handle the case when the path already exists. For that we will create another function that reads the current value of the database, and then sets a new value that is one greater than the current value.

```java
// This function updates the database by adding 1 to the count already in the database
private void updateDB(final DatabaseReference ref) {
    
    // Read the database once and update
    ref.addListenerForSingleValueEvent(new ValueEventListener() {
        @Override
        public void onDataChange(@NonNull DataSnapshot dataSnapshot) {
            final int value = dataSnapshot.getValue(Integer.class);
            ref.setValue(value + 1);
        }

        @Override
        public void onCancelled(@NonNull DatabaseError databaseError) {
            changeQuestionText("Could not update database");
        }
    });
}
```

We want to call this function where we previously wrote "Do something." We will also need to create the ref that we pass into our updateDB() function. The updated updateAndWatchDB() function is below.

```java
private void updateAndWatchDB(final String path) {
    DatabaseReference rootRef = FirebaseDatabase.getInstance().getReference();
    rootRef.addListenerForSingleValueEvent(new ValueEventListener() {
        @Override
        public void onDataChange(DataSnapshot snapshot) {
            final DatabaseReference ref = database.getReference(path);

            // Make sure that the path exists, if not, create it and set the value to 1
            if (snapshot.hasChild(path)) {
                updateDB(ref);
            }
            else {
                ref.setValue(1);  // If the path did not exist, then this is the first vote
            }
        }

        @Override
        public void onCancelled(DatabaseError error) {
            changeQuestionText("Could not read database");
        }
    });
}
```

## Watch for Data Changes

We have been working with a function named updateAndWatchDB(), but so far we have not done any watching. Firebase is a realtime database, which means that we can watch for changes in realtime and get notified if anything changes. In our case, we want to display the updated count of people who agree with us. The following function accomplishes that.

```java
// This function watches for changes to the ref and displays them
private void watchDB(final DatabaseReference ref) {

    ref.addValueEventListener(new ValueEventListener() {
        @Override
        public void onDataChange(DataSnapshot dataSnapshot) {
            final int value = dataSnapshot.getValue(Integer.class);
            changeQuestionText(value + " people agree!");
        }

        @Override
        public void onCancelled(DatabaseError error) {
            changeQuestionText("Could not read database");
        }
    });
}
```

Finally, we need to add it to our `updateAndWatchDB()` function. The final version of that function is shown below.

```java
// This function updates the database and then watches for changes
private void updateAndWatchDB(final String path) {
    DatabaseReference rootRef = FirebaseDatabase.getInstance().getReference();
    rootRef.addListenerForSingleValueEvent(new ValueEventListener() {
        @Override
        public void onDataChange(DataSnapshot snapshot) {
            final DatabaseReference ref = database.getReference(path);

            // Make sure that the path exists, if not, create it and set the value to 1
            if (snapshot.hasChild(path)) {
                updateDB(ref);
            }
            else {
                ref.setValue(1);  // If the path did not exist, then this is the first vote
            }

            watchDB(ref);
        }

        @Override
        public void onCancelled(DatabaseError error) {
            changeQuestionText("Could not read database");
        }
    });
}
```

## Testing

You can now run and test your app! Open your app and start voting on a picture. You should be see the vote count go up. Now look at your database in the Firebase Console and you should see your database updating. Try changing a number in the database and you should see the count on your app update. You should also see new keys appear as new pictures are voted on. 

You have successfully implemented a Firebase Realtime Database!!

# Resources

For more resources about firebase, visit

https://firebase.google.com/docs/guides/

https://firebase.google.com/docs/database/admin/retrieve-data

https://firebase.google.com/docs/auth/android/start/
