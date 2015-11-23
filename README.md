# PushStronger
杀死之后重新启动服务，接收离线消息的解决方案



public class AutoStarterReceiver extends BroadcastReceiver {

    private static final String TAG = "AutoStarter";

    @Override
    public void onReceive(Context context, Intent intent) {
        Log.d(TAG, "@onReceive");

        if (!AutoStarterService.isStart) {
            Log.d(TAG, "@onReceive NewsPushService will start");
            context.startService(new Intent(context, AutoStarterService.class));
        }
    }
}


public class AutoStarterService extends Service {
   
    private static final String TAG = "AutoStarterService";

    public static boolean isStart = false;

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        Log.d(TAG, "AutoStarterService onCreate");
        isStart = true;
        init();
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
       Log.d(TAG, "AutoStarterService onStartCommand");
        if (!isStart) {
           Log.d("AutoStarterService", "AutoStarterService onStartCommand ==> !isStart");
            init();
        }
        return super.onStartCommand(intent, flags, startId);        
    }

    @Override
    public void onDestroy() {
        isStart = false;
        // 添加自启动功能
        startAutoAlarm();
        
        super.onDestroy();
    }

    private void init(){
	//这块可以判断talk是否连接成功，然后如果没有，就启动下talk的服务
    }


    // 开启自启动闹钟
    private void startAutoAlarm() {
       //操作：发送一个广播, 启动AutoStarter
       Intent intent = new Intent(AutoStarterService.this, AutoStarterReceiver.class);
       intent.setAction("com.android.receiver.ACTION_AUTOSTARTER");
       PendingIntent autoStart = PendingIntent.getBroadcast(AutoStarterService.this, 0, intent, 0);

       //设定一个五秒后的时间
        Calendar calendar=Calendar.getInstance();
        calendar.setTimeInMillis(System.currentTimeMillis());
        calendar.add(Calendar.SECOND, 30);

       AlarmManager alarmManager = (AlarmManager) CApplication.getInstance().getSystemService(Context.ALARM_SERVICE);
       alarmManager.set(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(), autoStart);
    }

    //开启轮询服务
    public static void startPollingService(Context context, int seconds) {
        Intent intent = new Intent(context, AutoStarterReceiver.class);
        intent.setAction("com.android.receiver.ACTION_AUTOSTARTER");
        PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intent, 0);

        AlarmManager manager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
        //使用AlarmManger的setRepeating方法设置定期执行的时间间隔（seconds秒）和需要执行的Service
        manager.setRepeating(AlarmManager.ELAPSED_REALTIME, SystemClock.elapsedRealtime(), seconds * 1000, pendingIntent);
    }
}

//这句话表示每两分钟发送广播com.android.receiver.ACTION_AUTOSTARTER
接收者是AutoStarterReceiver
AutoStarterService.startPollingService(this, 2 * 60);


下面是放在androidmanifest里面的
接收广播android.net.conn.CONNECTIVITY_CHANGE网络变化，
com.android.receiver.ACTION_AUTOSTARTER 这是自己的
android.intent.action.USER_PRESENT 这是解锁的

<receiver android:name="com.android.receiver.AutoStarterReceiver" >
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
        <action android:name="com.android.receiver.ACTION_AUTOSTARTER" />
        <action android:name="android.intent.action.USER_PRESENT" />
    </intent-filter>
</receiver>

<service
        android:name="com.android.receiver.AutoStarterService"
        android:exported="true" >
</service>

