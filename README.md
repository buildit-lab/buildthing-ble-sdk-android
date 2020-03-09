# BuildThing™ ️beacon SDK for Android

[![BuildThing beacon](https://buildit.kr/dist/img/img-buildthing-beacon.ade36617.png)](https://buildit.kr/products/beacon-and-sdk)

BuildThing™ ️beacon SDK for Android는 Java로 구현된 Android용 SDK입니다.
Native [Android](https://developer.android.com/?hl=ko) 어플리케이션의 배포가 가능합니다.
SDK에서 제공하는 주요 기능들은 다음과 같습니다.

  - BuildThing™ beacon의 스캔 및 Advertising Packet 수신
  - BuildThing™ beacon 연결 및 비콘 값 설정
  - BuildThing™ beacon 영역의 진입/이탈 이벤트 수신 (Monitoring)
  - BuildThing™ beacon 과의 거리 확인 (Ranging)

## 목차
* [설치](#설치)
* [사용 가이드](#사용-가이드)
* [API 문서](#API-문서)
* [Scan Response](#Scan-Response)
* [Advertising Packet](#Advertising-Packet)
* [Connection Service](#Connection-Service)
* [참고 사항](#Connection-Service)
* [고객 문의](#고객-문의)

## 설치
BuildThing™ ️beacon SDK for Android의 실행을 위해서는 `Android API 18 버전 이상`을 필요로합니다.
`buildthing-ble-sdk-1.0.aar` 파일을 사용하고자 하는 프로젝트의 app/libs 폴더로 복사합니다.

## 사용 가이드
### 설정
#### build.gradle
App의 build.gradle 파일에 다음을 추가합니다.
```
repositories {
  flatDir {
    dirs 'libs'
  }
}
```
```
dependencies {
  ...
  implementation name: 'buildthing-ble-sdk-1.0', ext: 'aar'
  ...
}
```
#### ApplicationManifest.xml
Bluetooth 사용을 위해 ApplicationManifest.xml 파일에 다음을 추가합니다.
```
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
```
BLE의 작동을 위해 안드로이드 버전에 따라 다음을 추가합니다.
```
<!-- Android 9 이하일 경우 -->
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
<!-- Android 10 이상일 경우 -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<!-- 백그라운드 스캔을 사용하는 경우 -->
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
<!-- enableForegroundServiceScanning 사용하는 경우 -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```
#### 런타임에서 위치 권한 요청하기
아래와 같은 코드를 추가하여 실행하려는 앱에서 위치 정보 권한 활성화를 수행하여야합니다.
이는 안드로이드의 설정 -> 어플리케이션에서 SDK를 사용하는 어플리케이션의 권한 설정에서 수동으로 설정할 수도 있습니다.
##### Target SDK 버전 29 이상
```
public class MainActivity extends AppCompatActivity {
    protected static final String TAG = "MainActivity";
    private static final int PERMISSION_REQUEST_FINE_LOCATION = 1;
    private static final int PERMISSION_REQUEST_BACKGROUND_LOCATION = 2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            //위치 권한
            if (this.checkSelfPermission(Manifest.permission.ACCESS_FINE_LOCATION)
                    == PackageManager.PERMISSION_GRANTED) {
                //백그라운드 위치 권한
                if (this.checkSelfPermission(Manifest.permission.ACCESS_BACKGROUND_LOCATION)
                        != PackageManager.PERMISSION_GRANTED) {

                    if (this.shouldShowRequestPermissionRationale(Manifest.permission.ACCESS_BACKGROUND_LOCATION)) {
                        final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                        builder.setTitle("이 앱은 백그라운드 위치 접근 권한이 필요합니다.");
                        builder.setMessage("권한을 설정하여 이 앱이 백그라운드 비콘 스캔을 가능하게해주세요.");
                        builder.setPositiveButton(android.R.string.ok, null);
                        builder.setOnDismissListener(new DialogInterface.OnDismissListener() {

                            @TargetApi(23)
                            @Override
                            public void onDismiss(DialogInterface dialog) {
                                requestPermissions(new String[]{Manifest.permission.ACCESS_BACKGROUND_LOCATION},
                                        PERMISSION_REQUEST_BACKGROUND_LOCATION);
                            }

                        });
                        builder.show();
                    }
                    else {
                        final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                        builder.setTitle("권한 없음");
                        builder.setMessage("백그라운드 위치 접근 권한이 없습니다. 백그라운드에서 비콘 스캔이 불가능합니다. 설정 -> 어플리케이션 -> 권한에서 권한을 허용해주세요.");
                        builder.setPositiveButton(android.R.string.ok, null);
                        builder.setOnDismissListener(new DialogInterface.OnDismissListener() {

                            @Override
                            public void onDismiss(DialogInterface dialog) {
                            }

                        });
                        builder.show();
                    }

                }
            } else {
                if (this.shouldShowRequestPermissionRationale(Manifest.permission.ACCESS_FINE_LOCATION)) {
                    requestPermissions(new String[]{Manifest.permission.ACCESS_FINE_LOCATION,
                                    Manifest.permission.ACCESS_BACKGROUND_LOCATION},
                            PERMISSION_REQUEST_FINE_LOCATION);
                }
                else {
                    final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                    builder.setTitle("권한 없음");
                    builder.setMessage("위치 접근 권한이 없습니다. 비콘 스캔이 불가능합니다. 설정 -> 어플리케이션 -> 권한에서 위치 권한을 허용해주세요.");
                    builder.setPositiveButton(android.R.string.ok, null);
                    builder.setOnDismissListener(new DialogInterface.OnDismissListener() {

                        @Override
                        public void onDismiss(DialogInterface dialog) {
                        }

                    });
                    builder.show();
                }

            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
        switch (requestCode) {
            case PERMISSION_REQUEST_FINE_LOCATION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "위치 접근이 허용됨.");
                } else {
                    final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                    builder.setTitle("권한 없음");
                    builder.setMessage("위치 접근 권한이 없습니다. 비콘 스캔이 불가능합니다. 설정 -> 어플리케이션 -> 권한에서 위치 권한을 허용해주세요.");
                    builder.setPositiveButton(android.R.string.ok, null);
                    builder.setOnDismissListener(new DialogInterface.OnDismissListener() {

                        @Override
                        public void onDismiss(DialogInterface dialog) {
                        }

                    });
                    builder.show();
                }
                return;
            }
            case PERMISSION_REQUEST_BACKGROUND_LOCATION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "백그라운드 위치 접근 허용됨.");
                } else {
                    final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                    builder.setTitle("권한 없음");
                    builder.setMessage("백그라운드 위치 접근 권한이 없습니다. 백그라운드에서 비콘 스캔이 불가능합니다. 설정 -> 어플리케이션 -> 권한에서 권한을 허용해주세요.");
                    builder.setPositiveButton(android.R.string.ok, null);
                    builder.setOnDismissListener(new DialogInterface.OnDismissListener() {

                        @Override
                        public void onDismiss(DialogInterface dialog) {
                        }

                    });
                    builder.show();
                }
                return;
            }
        }
    }
}
```
##### Target SDK 버전 28 이하
```
public class MainActivity extends AppCompatActivity {
    protected static final String TAG = "MainActivity";
    private static final int PERMISSION_REQUEST_COARSE_LOCATION = 1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            // Android M Permission check 
            if (this.checkSelfPermission(Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
                final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                builder.setTitle("권한 없음");
                builder.setMessage("위치 접근 권한을 허용해야만 비콘을 스캔할 수 있습니다.");
                builder.setPositiveButton(android.R.string.ok, null);
                builder.setOnDismissListener(new DialogInterface.OnDismissListener() {
                    @Override
                    public void onDismiss(DialogInterface dialog) {
                        requestPermissions(new String[]{Manifest.permission.ACCESS_COARSE_LOCATION}, PERMISSION_REQUEST_COARSE_LOCATION);
                    }
                });
                builder.show();
            }
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode,
                                           String permissions[], int[] grantResults) {
        switch (requestCode) {
            case PERMISSION_REQUEST_COARSE_LOCATION: {
                if (grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                    Log.d(TAG, "위치 권한 허용됨.");
                } else {
                    final AlertDialog.Builder builder = new AlertDialog.Builder(this);
                    builder.setTitle("권한 없음");
                    builder.setMessage("위치 접근 권한을 허용해야만 비콘을 스캔할 수 있습니다.");
                    builder.setPositiveButton(android.R.string.ok, null);
                    builder.setOnDismissListener(new DialogInterface.OnDismissListener() {

                        @Override
                        public void onDismiss(DialogInterface dialog) {
                        }

                    });
                    builder.show();
                }
                return;
            }
        }
    }
}
```

### 비콘 스캔 예제 코드
```
public class ScanActivity extends Activity implements BLEServiceContext {
  protected static final String TAG = "ScanActivity";
    private Manager mManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_build_thing);
        mManager = Manager.getInstance(this);
        //디버깅 모드 ON
        mManager.setDebug(true);

        /*
            앱이 enableForegroundServiceScanning()을 통해 포그라운드서비스로 등록되어 있지 않을 때,
            비콘 스캔 수행 시, SDK는 기본적으로 JobScheduler를 사용하여 스캔하게 됩니다.
            이 때, 백그라운드 스캔은 최소 15분 간격으로만 스캔됩니다. (안드로이드 8.0, API 26 부터는 백그라운드에서 실행되는 앱 동작을 제한하였습니다.)
            만약 백그라운드에서도 원하는 만큼 더 짧은 간격의 스캔을 원하는 경우,
            enableForegroundServiceScanning 함수를 통해서 포그라운 서비스를 활성화하고 Notification 등록하여
            사용자가 이 앱이 백그라운드에서 실행되고 있음을 알릴 수 있어야합니다.
        */
        this.enableForegroundServiceScanning();
        mManager.bind(this);
    }

    public void enableForegroundServiceScanning() {
        // enableForegroundServiceScanning 를 사용하기 위한 NotificationChannel과 Notification 생성
        String CHANEL_ID = "channel_1";
        NotificationChannel channel = new NotificationChannel(CHANEL_ID, "Android test", NotificationManager.IMPORTANCE_LOW);
        ((NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE)).createNotificationChannel(channel);
        Notification notification = new NotificationCompat.Builder(this, CHANEL_ID).build();
        mManager.enableForegroundServiceScanning(notification, 1);
    }

    public void startScan(View view) throws RemoteException {
        // 스캔 시작
        mManager.startScan();
    }

    public void stopScan(View view) throws RemoteException {
        // 스캔 정지
        mManager.stopScan();
    }

    public void setForegroundScanPeriod(View view) throws RemoteException {
        EditText editText = (EditText)findViewById(R.id.foreground_scan_period);
        Long foregroundScanPeriod = 0l;
        try {
            foregroundScanPeriod = Long.parseLong(editText.getText().toString());
        } catch (NumberFormatException e) {}

        // 포그라운드 스캔의 지속 시간을 설정합니다.
        mManager.setForegroundScanPeriod(foregroundScanPeriod);
        mManager.updateScanPeriods();
    }

    public void setForegroundScanBetweenPeriod(View view) throws RemoteException {
        EditText editText = (EditText)findViewById(R.id.foreground_between_scan_period);
        Long foregroundScanBetweenPeriod = 0l;
        try {
            foregroundScanBetweenPeriod = Long.parseLong(editText.getText().toString());
        } catch (NumberFormatException e) {}

        // 포그라운드 스캔 사이클의 주기를 설정합니다.
        mManager.setForegroundBetweenScanPeriod(foregroundScanBetweenPeriod);
        mManager.updateScanPeriods();
    }

    public void setBackgroundScanPeriod(View view) throws RemoteException {
        EditText editText = (EditText)findViewById(R.id.background_scan_period);
        Long backgroundScanPeriod = 0l;
        try {
            backgroundScanPeriod = Long.parseLong(editText.getText().toString());
        } catch (NumberFormatException e) {}

        // 백그라운드 스캔의 지속 시간을 설정합니다.
        mManager.setBackgroundScanPeriod(backgroundScanPeriod);
        mManager.updateScanPeriods();
    }

    public void setBackgroundBetweenScanPeriod(View view) throws RemoteException {
        EditText editText = (EditText)findViewById(R.id.background_between_scan_period);
        Long backgroundBetweenScanPeriod = 0l;
        try {
            backgroundBetweenScanPeriod = Long.parseLong(editText.getText().toString());
        } catch (NumberFormatException e) {}

        // 백그라운드 스캔 사이클의 주기를 설정합니다.
        mManager.setBackgroundBetweenScanPeriod(backgroundBetweenScanPeriod);
        mManager.updateScanPeriods();
    }

    @Override
    public void onBLEServiceConnect () {
        Log.i(TAG, "service connected");
        // 스캔 이벤트 리스너 초기화
        mManager.removeAllScanListener();
        mManager.addScanListener(new ScanListener() {
            @Override
            public void onDiscover(Beacon beacon) {
                // 스캔중 발견한 비콘 인스턴스를 인자로 전달받습니다.
                Log.i(TAG, "onDiscover beacon: " + beacon);
            }

            @Override
            public void onUndiscover(List<Beacon> beacons) {
                // 설정한 스캔 주기 동안 발견되지 않은 비콘 인스턴스를 인자로 전달받습니다.
                Log.i(TAG, "onUndiscover beacons: " +  beacons);
            }
        });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mManager.unbind(this);
    }
}
```
### Monitoring / Ranging 예제 코드
```
public class MonitoringAndRangingActivity extends Activity implements BLEServiceContext {
  protected static final String TAG = "MonitoringAndRangingActivity";
    private Manager mManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_build_thing);
        mManager = Manager.getInstance(this);
        //디버깅 모드 ON
        mManager.setDebug(true);

        /*
            앱이 enableForegroundServiceScanning()을 통해 포그라운드서비스로 등록되어 있지 않을 때,
            비콘 스캔 수행 시, SDK는 기본적으로 JobScheduler를 사용하여 스캔하게 됩니다.
            이 때, 백그라운드 스캔은 최소 15분 간격으로만 스캔됩니다. (안드로이드 8.0, API 26 부터는 백그라운드에서 실행되는 앱 동작을 제한하였습니다.)
            만약 백그라운드에서도 원하는 만큼 더 짧은 간격의 스캔을 원하는 경우,
            enableForegroundServiceScanning 함수를 통해서 포그라운 서비스를 활성화하고 Notification 등록하여
            사용자가 이 앱이 백그라운드에서 실행되고 있음을 알릴 수 있어야합니다.
        */
        this.enableForegroundServiceScanning();
        mManager.bind(this);
    }

    public void enableForegroundServiceScanning() {
        // enableForegroundServiceScanning 를 사용하기위해 적절한 채널과, Notification 을 생성해야 합니다.
        String CHANEL_ID = "channel_1";
        NotificationChannel channel = new NotificationChannel(CHANEL_ID, "Android test", NotificationManager.IMPORTANCE_LOW);
        ((NotificationManager)getSystemService(Context.NOTIFICATION_SERVICE)).createNotificationChannel(channel);
        Notification notification = new NotificationCompat.Builder(this, CHANEL_ID).build();
        mManager.enableForegroundServiceScanning(notification, 1);
    }

    public void startMonitoring(View view) throws RemoteException {
        // 새로운 모니터링 영역을 추가하고, 해당 영역의 모니터링을 시작합니다.
        mManager.startMonitoring(new Region("MyRegion", 1111, 1111, "", ""));
    }

    public void stopMonitoring(View view) throws RemoteException {
        // 해당 모니터링 영역을 제거하고, 해당 영역의 모니터링을 정지합니다.
        mManager.stopMonitoring(new Region("MyRegion", 1111, 1111, "", ""));
    }

    public void startRanging(View view) throws RemoteException {
        // 새로운 레인징 영역을 추가하고, 해당 영역의 레인징을 시작합니다.
        mManager.startRanging(new Region("MyRegion", 1111, 1111, "", ""));
    }

    public void stopRanging(View view) throws RemoteException {
        // 해당 레인징 영역을 제거하고, 해당 영역의 레인징을 정지합니다.
        mManager.stopRanging(new Region("MyRegion", 1111, 1111, "", ""));
    }

    @Override
    public void onBLEServiceConnect () {
      // 모니터링 이벤트 리스너 초기화
      mManager.removeAllMonitoringListener();
      mManager.addMonitoringListener(new MonitoringListener(){
          @Override
          public void onEnter(Region region) {
              // 영역과 동일한 식별 값(Major, Minor 등)을 갖는 비콘 반경에 진입한 경우, 해당 영역을 인자로 전달받습니다.
              Log.i(TAG, "onEnter region: " +  region);
          }

          @Override
          public void onExit(Region region) {
              // 영역과 동일한 식별 값(Major, Minor 등)을 갖는 비콘이 더 이상 감지 되지 않는 경우, 해당 영역을 인자로 전달받습니다.
              Log.i(TAG, "onExit region: " + region));
          }

          @Override
          public void onUpdate(int state, Region region) {
              // 모니터링하고 있는 영역 중에 진입/이탈 상태가 업데이트 될 경우, 업데이트 상태와 해당 영역을 인자로 전달받습니다. (1: Enter, 2: Exit)
              Log.i(TAG, "onUpdate state: " + state);
              Log.i(TAG, "onUpdate region: " + region);
          }
      });

      // 레인징 이벤트 리스너 초기화
      mManager.removeAllRangeListener();
      mManager.addRangeListener(new RangeListener() {
          @Override
          public void onRanged(Collection<Beacon> beacons, Region region) {
              // 레인징되고 있는 영역과 동일한 식별 값(Major, Minor 등)을 갖는 비콘들의 거리가 계산 되고, 해당 영역과 영역에 해당하는 비콘들을 인자로 전달받습니다.
              Log.i(TAG, "onRanged beacons: " + beacons);
              Log.i(TAG, "onRanged region: " + region);
              for (Beacon beacon : beacons) {
                  Log.i(TAG, "onRanged pass beacon: " + beacon.getName() + "/ distance: " + beacon.getDistance());
              }
          }
      });
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mManager.unbind(this);
    }
}
```
### Connection 예제 코드
```
public class ConnectionActivity extends Activity implements BLEServiceContext {
    protected static final String TAG = "ConnectionActivity";
    private Manager mManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mManager = Manager.getInstance(this);
        mManager.bind(this);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        mManager.unbind(this);
    }

    @Override
    public void onBLEServiceConnect() {
        mManager.removeAllScanListener();
        mManager.addScanListener(new ScanListener() {
            @Override
            public void onDiscover(Beacon beacon) {
                try {
                    // 발견된 비콘에 연결을 시도합니다.
                    mManager.connectToBeacon(beacon);
                } catch (RemoteException e) {}
            }

            @Override
            public void onUndiscover(List<Beacon> beacons) {
            }
        });
        mManager.removeAllConnectionListener();
        mManager.addConnectionListener(new ConnectionListener() {
            @Override
            public void onConnect(ConnectedBeacon connectedBeacon) {
                //연결 성공 콜백
                LogManager.i(TAG, "Beacon %s is connected", connectedBeacon.getBeacon().getName());
                // 전달받은 ConnectedBeacon 인스턴스에 변경할 속성 이름과 값을 지정합니다.
                connectedBeacon.setCommandName(Connection.COMMAND_CHANGE_NAME);
                connectedBeacon.setValue(new String[] {"New name"});
                try {
                    // 매니저에 속성 변경을 요청합니다.
                    mManager.changeBeaconValue(connectedBeacon);
                } catch (RemoteException e) {}
            }

            @Override
            public void onDisconnect(Beacon beacon) {
                // 연결 해제 콜백
                LogManager.i(TAG, "Beacon %s is disconnected", beacon.getName());
            }

            @Override
            public void onChanged(Beacon beacon, boolean success) {
                // 값 변경 성공 콜백
                LogManager.i(TAG, "Beacon %s's value change %s", beacon.getName(), success);
                try {
                    // 값 변경이 끝난 비콘을 연결 해제합니다.
                    mManager.disconnectFromBeacon(beacon);
                } catch (RemoteException e) {}
            }

            @Override
            public void onConnectionError(ConnectionException connectionException) {
                // 커넥션이 비정상 적으로 해제될 경우 호출됩니다. 비정상 적으로 해제되는 경우는 다음과 같습니다.
                // 커넥션 요청을 하였으나, 정상적으로 연결되지 않는 경우(connection timeout)
                // 값 변경을 시도 하였으나, 연결이 되지 않는 경우(beacon connection was destroyed. retry connect)
                LogManager.i(TAG, "Cannot connect to %s because of %s", beacon.getName(), connectionException.getBeacon(), connectionException.getMessage());
            }

            @Override
            public void onChangedError(ConnectionChangedException connectionChangedException) {
                // 값 변경이 실패한 경우 호출됩니다.
                // 값이 유효하지 않는 경우는 InvalidValueException 이 발생합니다.
                // 비콘에서 값 변경이 실패한 경우는 ValueChangeRejectedException 이 발생합니다.
                LogManager.i(TAG, "Cannot change %s's value because of %s", beacon.getName(), connectionChangedException.getBeacon(), connectionChangedException.getMessage());
            }
        });
        try {
            mManager.startScan();
        } catch (RemoteException e) {}

    }
}
```
### 백그라운드에서의 앱 실행
백그라운드 스캔 중 모니터링 중인 영역 발견 시, 앱의 특정 Activity가 실행될 수 있게 하는 예제 코드입니다.
`해당 기능을 사용하기 위해서는 최소한 한번은 사용자가 앱을 실행시켜놓은 상태여야하며, 앱이 SD 카드가 아닌 내부 메모리에 설치되어있어야합니다.'`

AndroidManifest.xml 설정
```
<application
        android:name="com.example.MyRegionApp"
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        <!-- Note: 아래에 launchMode 값을 singleInstance 로 설정하지 않으면 두 개의 인스턴스가 생기게 됩니다. -->
        <activity
            android:launchMode="singleInstance"
            android:name="com.example.MainActivity"
            android:label="@string/app_name" >
            <intent-filter>
        <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
```

실행 예제
```
public class MyRegionApp extends Application implements ApplicationListener {
    private static final String TAG = ".MyRegionApp";
    private RegionInApplication mRegion;

    @Override
    public void onCreate() {
        super.onCreate();
        // 모니터링 하려는 영역을 추가합니다.
        Region region = new Region("MyRegion", 1111, 1111, "", "");
        mRegion = new RegionInApplication(this, region);
    }

    @Override
    public void onEnter(Region region) {
        //비활성화하여 최초 비콘이 발견되었을 때만 호출되게 합니다.
        mRegion.disable();
        // 영역이 발견되면 MainActivity 화면이 자동으로 진입하도록 합니다.
        Intent intent = new Intent(this, MainActivity.class);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        this.startActivity(intent);
    }

    @Override
    public void onExit(Region region) {}

    @Override
    public void onUpdate(int state, Region region) {}
}
```

### 배터리 절약 모드
```
public class BleBackgroundPowerSaverApp extends Application {
    private BackgroundPowerSaver backgroundPowerSaver;
    @Override
    public void onCreate() {
        super.onCreate();
        // BackgroundPowerSaver 인스턴스를 생성하면서 Application class를 전달해놓으면,
        // 자동으로 배터리 약 60% 의 배터리 절약을 기대할 수 있습니다.
        backgroundPowerSaver = new BackgroundPowerSaver(this);
    }
}
```

## API 문서
상세한 API는 아래 문서를 참고해주시기 바랍니다.
[API 문서 바로가기](https://buildit-lab.github.io/buildthing-beacon-sdk-android/)

## Scan Response
BuildThing beacon 스캔 수행 시, Scan Response로 수신되는 Service UUID와 Service 별 수신 값은 아래와 같습니다.
아래 Scan Response는 BuildThing, iBeacon, Eddystone 모드 모두 동일합니다.

| 항목 | Service UUID | 수신 값 | 비고 |
| ------ | ------ | ------ |------ |
| Device Information & Firmware Version | 0x180A | 0x983006 또는 0x3006XX | 0x983006 : 펌웨어 v1.0, 0x3006XX : 펌웨어 vXX/10 (예 : 0x300611은 펌웨어 v1.1)
| Mac Address | 0xADD0 | Mac Address | - |

## Advertising Packet
BuildThing beacon은 BuildThing 모드 외 iBeacon, Eddystone 모드의 Advertising Packet 을 지원합니다. Advertising Packet을 수신하기 위한 각 모드 별 UUID 및 Service UUID는 다음과 같습니다.

| 비콘 모드 | UUID | Service UUID |
| ------ | ------ | ------ |
| BuildThing | - | 0xBCBC |
| iBeacon | 0x0b2b0848-205f-11e9-ab14-820316983006 | - |
| Eddystone | - | 0xFEAA |

비콘 모드 별 수신되는 Advertising Packet Format은 아래와 같습니다.
[![BuildThing beacon Advertising Packet Format](https://buildit.kr/dist/assets/buildthing-beacon-advertising-packet-format.png)](https://buildit.kr/dist/assets/buildthing-beacon-advertising-packet-format.png)

## Connection Service
아래 Connection Service UUID를 통하여 BuildThing beacon Connection Service에 연결합니다.

| 항목 | Service UUID |
| ------ | ------ |
| Connection Service | 6E400001-B5A3-F393-E0A9-E50E24DCCA9E |

Connection이 완료되면 아래 2개의 Characteristic에 접근할 수 있습니다.

| 항목 | Characteristic UUID | 속성 | 비고 |
| ------ | ------ | ------ | ------ |
| 비콘 값 설정 | 6E400002-B5A3-F393-E0A9-E50E24DCCA9E | Read, Write | 비콘 값 을 설정할 수 있습니다.
| 비콘 값 설정 결과 | 6E400003-B5A3-F393-E0A9-E50E24DCCA9E | Notify | 비콘 값 설정 완료 후 Notify를 수신할 수 있습니다.

비콘 값 설정 Characteristic에 값을 Write하여 비콘 값을 설정합니다.
설정 시, 아래와 같은 전송 패킷 포맷으로 Write 합니다.
**최초 Password 값은 000000** 입니다.

[![BuildThing beacon Connection Service](https://buildit.kr/dist/assets/buildthing-beacon-connection-service.png)](https://buildit.kr/dist/assets/buildthing-beacon-connection-service.png)

## 참고 사항
### 거리 계산 테스트
비콘과 기기 간의 거리 계산에 대하여 테스트가 완료된 기기 모델은 아래와 같습니다.

| 플랫폼 | 모델명 |
| ------ | ------ |
| Android | Galaxy S8, Galaxy S9, Galaxy S10, Galaxy Note 9|

## 고객 문의
Github Issue 외 기타 문의 사항은 web.dev@buildit.kr로 문의해주시기 바랍니다.
