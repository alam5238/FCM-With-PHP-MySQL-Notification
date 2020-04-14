# FCM-With-PHP-MySQL-Notification
FCM Notification with PHP &amp; MySQL for Android

**Step 1**
   Add dependencies in `build.gradle(Module)` and
   
   ```
   dependencies {
    implementation 'com.google.firebase:firebase-messaging:20.1.5'
   }

   ```
   
   add dependencies classpath in `build.gradle(Project)` of your project. 
   ```
   dependencies {
       
      classpath 'com.google.gms:google-services:4.3.3'
       
   }
   ```
**Step 2**
   Add Service to `AndroidMAinfest.xml` in `<application></application>` tag.
   ```
   <application>
     <service
            android:name=".FirebaseMessingService"
            android:exported="false">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT" />
            </intent-filter>
      </service>
   </application>
   
   ```
   
**Step 3**
  Create `FirebaseMessingService` java class file in java directory 
  
  In `FirebaseMessingService` java file extends FirebaseMessagingService

```
public class FirebaseMessingService extends FirebaseMessagingService {

        private static final String TAG = "0";


    @Override
    public void onMessageReceived(@NonNull RemoteMessage remoteMessage) {
        String msg = remoteMessage.getData().get("message");  //recive broadcast notification
            showNotification(msg);    // method for call notification
        Log.d(TAG,"RECIVE : "+msg);   
    }



    private void showNotification(String message) {

        Intent i = new Intent(getBaseContext(),Notification_Show.class);
        i.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        i.putExtra("Title", message);

        PendingIntent pendingIntent = PendingIntent.getActivity(getBaseContext(),0,i,PendingIntent.FLAG_UPDATE_CURRENT);


        NotificationManagerCompat notificationManager = NotificationManagerCompat.from(getApplicationContext());
        Notification notification = new NotificationCompat.Builder(this, CHANNEL_1_ID)
                .setSmallIcon(R.drawable.ic_directions_bike_black_24dp)
                .setContentTitle("Title of Notification")
                .setContentText(message)
                .setContentIntent(pendingIntent)
                .setPriority(NotificationCompat.PRIORITY_HIGH)
                .setCategory(NotificationCompat.CATEGORY_MESSAGE)
                .setAutoCancel(true)
                .build();

        notificationManager.notify(1, notification);



    }

    @Override
    public void onNewToken(@NonNull String s) {
        Log.d(TAG,"RECIVE : Token - "+s);
        storeToken(s);  // method for storing token on Database Server
    }
    
    
    RequestQueue mQueue = VollySingleton.getInstance(this).getmRequestQueue();

    private void storeToken(String s) {
        String url = "http://192.168.0.107/fcm/registration.php?Token="+s;

        JsonObjectRequest request = new JsonObjectRequest(Request.Method.GET, url, null,
                new Response.Listener<JSONObject>() {
                    @Override
                    public void onResponse(JSONObject response) {
                        try {
                            JSONArray jsonArray = response.getJSONArray("employees");

                            for (int i = 0; i < jsonArray.length(); i++) {

                                Log.d(TAG,"RECIVE : Token Stored - OK");
                            }
                        } catch (JSONException e) {
                            e.printStackTrace();
                        }
                    }
                }, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                error.printStackTrace();
            }
        });

        mQueue.add(request);


    }





}
```

Here, `onMessageReceived()` method recive notification data from server using  `FirebaseMessagingService` and the method `onNewToken()` 
generate a token of your new device.This token generate when divice change or uninstall/install application or clean app data.
`storeToken()` method store generate token to server via JSON Volly library Request.(You can use other http request library for store token in server). And the last method is `showNotification()` that build and show notification when and massege or notification recive.


**Step4**
For showing notification first create a notification channel. For creating notification channel, create `App` java class file in java directory. The `App` class file extends Application. 

```
public class App extends Application {

    public static final String CHANNEL_1_ID = "channel1";
    public static final String CHANNEL_2_ID = "channel2";

    @Override
    public void onCreate() {
        super.onCreate();

        createNotificationChannels();
    }


    private void createNotificationChannels() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            NotificationChannel channel1 = new NotificationChannel(
                    CHANNEL_1_ID,
                    "Channel 1",
                    NotificationManager.IMPORTANCE_HIGH
            );
            channel1.setDescription("This is Channel 1");

            NotificationChannel channel2 = new NotificationChannel(
                    CHANNEL_2_ID,
                    "Channel 2",
                    NotificationManager.IMPORTANCE_LOW
            );
            channel2.setDescription("This is Channel 2");

            NotificationManager manager = getSystemService(NotificationManager.class);
            manager.createNotificationChannel(channel1);
            manager.createNotificationChannel(channel2);
        }
    }

}
```
For register `App` class add `android:name=.App` in AndroidMainfest.xml file `<application>` tag.



