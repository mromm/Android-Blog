
### Android开发经常会用到notification通知，主要用来提示用户有新消息、推送消息或者音乐播放进入后台通知栏显示播放进度等，使用通知会大大提高用户的体验

#### notification是系统通知，我们可以使用系统默认的样式和图标显示，当然也可以通过自定义样式，默认样式使用比较简单，这里就使用自定义样式来讲解说明,直接上代码：  
```
//获取系统通知服务的管理类，要想开启通知，必须要要获取管理者
 NotificationManager nm = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);

        NotificationCompat.Builder builder = new NotificationCompat.Builder(context);//获取通知的构造者，用来添加自定义样式
        //创建自定义样式，和普通的view创建方法不一样，注意!!!
        RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.notifacation_player_view);

        if (isPlay) {
            remoteViews.setImageViewResource(R.id.notify_play, R.drawable.btn_pause_selector);
        } else {
            remoteViews.setImageViewResource(R.id.notify_play, R.drawable.btn_play_selector);
        }
        //给布局中的控件重新赋值，查看源码发现使用反射调用控件的set方法，相当直接使用控件来进行赋值
        remoteViews.setTextViewText(R.id.notify_title, title);

        //设置播放或暂停意图
        Intent play = new Intent();

        play.setAction(Constant.NOTIFY_PLAY_PAUSE);
        //给布局中的控件添加意图，也就是添加事件，PendingIntent是一种特殊的intent，主要用在通知中
        //注意这里使用的是getBroadcast(),将我们的意图通过广播发送出去，以便能在页面中接受并处理
        PendingIntent playPendingIntent = PendingIntent.getBroadcast(context, 0, play, PendingIntent.FLAG_UPDATE_CURRENT);

        //设置上一首意图
        Intent pre = new Intent();
        pre.setAction(Constant.NOTIFY_PRE);
        PendingIntent prePendingIntent = PendingIntent.getBroadcast(context, 1, pre, PendingIntent.FLAG_UPDATE_CURRENT);

        //设置下一首意图
        Intent next = new Intent();
        next.setAction(Constant.NOTIFY_NEXT);
        PendingIntent nextPendingIntent = PendingIntent.getBroadcast(context, 2, next, PendingIntent.FLAG_UPDATE_CURRENT);


        Intent cancel = new Intent();
        cancel.setAction(Constant.NOTIFY_CANCEL);

        PendingIntent closePendingIntent = PendingIntent.getBroadcast(context, 5, cancel, PendingIntent.FLAG_UPDATE_CURRENT);

        //给控件添加点击事件，将上面创建的意图传进来
        remoteViews.setOnClickPendingIntent(R.id.notify_play, playPendingIntent);
        remoteViews.setOnClickPendingIntent(R.id.notify_prev, prePendingIntent);
        remoteViews.setOnClickPendingIntent(R.id.notify_next, nextPendingIntent);

        remoteViews.setOnClickPendingIntent(R.id.notify_close, closePendingIntent);

        Intent intent = new Intent(context, ListenEnglishActivity.class);

        //点击整个通知处理意图，这里使用的是activity,直接跳转到界面中去，注意跳转的activity必须是signatask启动模式，这里我在清单文件中指定了
        PendingIntent notificationPendingIntent = PendingIntent.getActivity(context, 6, intent, PendingIntent.FLAG_UPDATE_CURRENT);

        builder.setCustomBigContentView(remoteViews)//设置大图标通知
                .setCustomContentView(remoteViews)//设置普通通知
                .setContentIntent(notificationPendingIntent);//设置通知点击的意图
        Notification notification = builder.build();
        notification.tickerText = "播放通知";//通知的tickerText和icon必不可少，否则通知不会显示，切记!!!
        notification.icon = R.mipmap.lanucher;
        notification.flags |= flags;//设置通知的清理方式，自动取消、无法清理等

        nm.notify(100, notification);//发布通知
        ```
        上面一段是创建通知的过程，接下来还需要注册广播来处理通知中各种事件：
        ```
        //动态注册广播
         private void registerReciver() {
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Constant.NOTIFY_NEXT);
        intentFilter.addAction(Constant.NOTIFY_PRE);
        intentFilter.addAction(Constant.NOTIFY_PLAY_PAUSE);
        intentFilter.addAction(Constant.NOTIFY_CANCEL);
        receiver = new NotificationPlayerReceiver();
        this.registerReceiver(receiver, intentFilter);
    }

    public class NotificationPlayerReceiver extends BroadcastReceiver {

        private static final String TAG = "NotificationPlayerRecei";
        //对广播中接受action做相应的处理
        @Override
        public void onReceive(Context context, Intent intent) {
            String action = intent.getAction();
            Log.e(TAG, "onReceive: " + action);
            if (!TextUtils.isEmpty(action)) {
                switch (action) {
                    case Constant.NOTIFY_NEXT:
                        next();
                        break;
                    case Constant.NOTIFY_PRE:
                        pre();
                        break;
                    case Constant.NOTIFY_PLAY_PAUSE:
                        mLyricViewPresenter.onBtnPlayPausePressed();
                        break;
                    case Constant.NOTIFY_CANCEL:
                        NotificationUtil.clear(ListenEnglishActivity.this);
                        break;
                }
            }
        }
    }
    ```

