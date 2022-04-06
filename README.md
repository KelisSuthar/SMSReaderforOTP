# SMSReaderforOTP# Welcome To SMS Reader (For OTP you can get OTP from sms)

**Steps To implement The sms Preview in this(Almost Same for java)**

**Note - I have refer this site to do this project and guid youe acordingly**

=> Create New android project name SMS Reader

=> Implment below dependancy into your poject in build.gradle file

    ```java
    implementation 'com.google.android.gms:play-services-auth:20.1.0'
    implementation 'com.google.android.gms:play-services-auth-api-phone:18.0.1'
    ```

Let the project sync

=> Listen to the incoming messages

    ```Java
      private void startSmsUserConsent() {
    SmsRetrieverClient client = SmsRetriever.getClient(this);
    //We can add sender phone number or leave it blank
    // I'm adding null here
    client.startSmsUserConsent(null).addOnSuccessListener(new OnSuccessListener<Void>() {
      @Override
      public void onSuccess(Void aVoid) {
        Toast.makeText(getApplicationContext(), "On Success", Toast.LENGTH_LONG).show();
      }
    }).addOnFailureListener(new OnFailureListener() {
      @Override
      public void onFailure(@NonNull Exception e) {
        Toast.makeText(getApplicationContext(), "On OnFailure", Toast.LENGTH_LONG).show();
      }
    });
    }```

=>Create New Class for Brodcastreciever

    ```java
        SmsBroadcastReceiverListener smsBroadcastReceiverListener;
        ```

    ```java

        SmsBroadcastReceiverListener smsBroadcastReceiverListener;
        @Override
        public void onReceive(Context context, Intent intent) {
            if (intent.getAction() == SmsRetriever.SMS_RETRIEVED_ACTION) {
            Bundle extras = intent.getExtras();
            Status smsRetrieverStatus = (Status) extras.get(SmsRetriever.EXTRA_STATUS);
            switch (smsRetrieverStatus.getStatusCode()) {
                case CommonStatusCodes.SUCCESS:
                Intent messageIntent = extras.getParcelable(SmsRetriever.EXTRA_CONSENT_INTENT);
                smsBroadcastReceiverListener.onSuccess(messageIntent);
                break;
                case CommonStatusCodes.TIMEOUT:
                smsBroadcastReceiverListener.onFailure();
                break;
            }
            }
        }
        public interface SmsBroadcastReceiverListener {
            void onSuccess(Intent intent);
            void onFailure();
        }
    
    ```

=>Show the permission consent

    ```java

    private void registerBroadcastReceiver() {
    smsBroadcastReceiver = new SmsBroadcastReceiver();
    smsBroadcastReceiver.smsBroadcastReceiverListener =
        new SmsBroadcastReceiver.SmsBroadcastReceiverListener() {
          @Override
          public void onSuccess(Intent intent) {
            startActivityForResult(intent, REQ_USER_CONSENT);
          }

          @Override
          public void onFailure() {

          }
        };
    IntentFilter intentFilter = new IntentFilter(SmsRetriever.SMS_RETRIEVED_ACTION);
    registerReceiver(smsBroadcastReceiver, intentFilter);
     }

    @Override
     protected void onStart() {
    super.onStart();
        registerBroadcastReceiver();
    }

    @Override
    protected void onStop() {
    super.onStop();
    unregisterReceiver(smsBroadcastReceiver);
    }
    ```

=>Receive message in onActivityResult()

    ```java

        @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQ_USER_CONSENT) {
        if ((resultCode == RESULT_OK) && (data != null)) {
            //That gives all message to us.
            // We need to get the code from inside with regex
            String message = data.getStringExtra(SmsRetriever.EXTRA_SMS_MESSAGE);
            Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
            textViewMessage.setText(
                String.format("%s - %s", getString(R.string.received_message), message));
            getOtpFromMessage(message);
        }
        }
    }
    ```
=>Extract OTP from messaging.

    ```java
          private void getOtpFromMessage(String message) {
    // This will match any 6 digit number in the message
    Pattern pattern = Pattern.compile("(|^)\\d{6}");
    Matcher matcher = pattern.matcher(message);
    if (matcher.find()) {
      otpText.setText(matcher.group(0));
    }
    }
    ```

=>Final Combine Code

    ```java
    public class MainActivity extends AppCompatActivity {
    private static final int REQ_USER_CONSENT = 200;
    SmsBroadcastReceiver smsBroadcastReceiver;
    Button verifyOTP;
    TextView textViewMessage;
    EditText otpText;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        // find view by ids
        verifyOTP = findViewById(R.id.button);
        textViewMessage = findViewById(R.id.textViewMessage);
        otpText = findViewById(R.id.editTextOTP);
        startSmsUserConsent();
    }
    private void startSmsUserConsent() {
        SmsRetrieverClient client = SmsRetriever.getClient(this);
        //We can add sender phone number or leave it blank
        // I'm adding null here
        client.startSmsUserConsent(null).addOnSuccessListener(new OnSuccessListener<Void>() {
        @Override
        public void onSuccess(Void aVoid) {
            Toast.makeText(getApplicationContext(), "On Success", Toast.LENGTH_LONG).show();
        }
        }).addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) {
            Toast.makeText(getApplicationContext(), "On OnFailure", Toast.LENGTH_LONG).show();
        }
        });
    }
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == REQ_USER_CONSENT) {
        if ((resultCode == RESULT_OK) && (data != null)) {
            //That gives all message to us.
            // We need to get the code from inside with regex
            String message = data.getStringExtra(SmsRetriever.EXTRA_SMS_MESSAGE);
            Toast.makeText(getApplicationContext(), message, Toast.LENGTH_LONG).show();
            textViewMessage.setText(
                String.format("%s - %s", getString(R.string.received_message), message));
            getOtpFromMessage(message);
        }
        }
    }
    private void getOtpFromMessage(String message) {
        // This will match any 6 digit number in the message
        Pattern pattern = Pattern.compile("(|^)\\d{6}");
        Matcher matcher = pattern.matcher(message);
        if (matcher.find()) {
        otpText.setText(matcher.group(0));
        }
    }
    private void registerBroadcastReceiver() {
        smsBroadcastReceiver = new SmsBroadcastReceiver();
        smsBroadcastReceiver.smsBroadcastReceiverListener =
            new SmsBroadcastReceiver.SmsBroadcastReceiverListener() {
            @Override
            public void onSuccess(Intent intent) {
                startActivityForResult(intent, REQ_USER_CONSENT);
            }
            @Override
            public void onFailure() {
            }
            };
        IntentFilter intentFilter = new IntentFilter(SmsRetriever.SMS_RETRIEVED_ACTION);
        registerReceiver(smsBroadcastReceiver, intentFilter);
    }
    @Override
    protected void onStart() {
        super.onStart();
        registerBroadcastReceiver();
    }
    @Override
    protected void onStop() {
        super.onStop();
        unregisterReceiver(smsBroadcastReceiver);
    }
    }
        ```

**Thankyou**
