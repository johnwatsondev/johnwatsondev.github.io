---
title: 检查运行时权限工具类
layout: post
guid: urn:uuid:f9b93732-9806-4fd1-ba4b-4705a0743cd3
comments: true
tags:
  - permission
---

作者：[JohnWatsonDev](http://johnwatsondev.com)  
转载请注明出处 --- 有节操工程师必备品质~

### 前言
在 6.0 上使用 [PhotoPicker](https://github.com/donglua/PhotoPicker) ，由于默认情况下应用权限是未通过的，所以出现问题。  

### 异常
```java
Caused by: java.lang.SecurityException: Permission Denial:  
reading com.android.providers.media.MediaProvider uri content://media/external/images/media  
requires android.permission.READ_EXTERNAL_STORAGE, or grantUriPermission()
```

显然是木有权限呀，一番查证后是 [Runtime Permissions](http://developer.android.com/training/permissions/requesting.html) 问题。参考文档后抽取工具类如下：

```java
/**
 * 检查权限工具类
 * Created by johnwatson on 11/17/15.
 */
public class PermissionUtil {

  private static final String TAG = PermissionUtil.class.getSimpleName();

  /**
   * 如果需要说明请求权限的原因，将回调该接口。
   */
  public interface ShowRationaleCallback {
    /**
     * Call this method if we should show request permission rationale.
     */
    void needShow(final int requestCode);
  }

  /**
   * 检查权限是否已获取
   *
   * @param activity 请求权限要求必须获取 @link{android.app.Activity} 对象
   * @param permission 目标权限
   * @param requestCode 请求权限的请求码
   * @param callback 当需要给用户显示提示信息时回调函数
   * @param needRequestPermission 当需要给用户显示提示信息时，是否直接帮开发者请求权限。
   * @return true if granted
   */
  public static boolean checkPermissionGranted(final Activity activity, final String permission,
      final int requestCode, final ShowRationaleCallback callback,
      final boolean needRequestPermission) {
    boolean granted = true;

    // Here, thisActivity is the current activity
    if (ContextCompat.checkSelfPermission(activity, permission)
        != PackageManager.PERMISSION_GRANTED) {
      granted = false;

      boolean shouldShowRationale = false;

      // Should we show an explanation?
      if (ActivityCompat.shouldShowRequestPermissionRationale(activity, permission)) {
        // Show an expanation to the user *asynchronously* -- don't block
        // this thread waiting for the user's response! After the user
        // sees the explanation, try again to request the permission.

        if (callback != null) {
          callback.needShow(requestCode);
        }

        shouldShowRationale = true;
      } else {
        // No explanation needed, we can request the permission.
        // MY_PERMISSIONS_REQUEST_READ_CONTACTS is an
        // app-defined int constant. The callback method gets the
        // result of the request.
      }

      if (BuildConfig.DEBUG) {
        Log.d(TAG, "shouldShowRequestPermissionRationale = " + shouldShowRationale);
      }

      if (!shouldShowRationale || needRequestPermission) {
        ActivityCompat.requestPermissions(activity, new String[] { permission }, requestCode);
      }
    }

    return granted;
  }

  /**
   * 检查权限请求返回的结果
   *
   * @param grantResults 请求结果
   * @return true if granted
   */
  public static boolean isGrantedForResult(int[] grantResults) {
    return grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED;
  }
}
```

<!-- {% gist johnwatsondev/2d9a068931f88a3b2470%} -->

### 解释

重点解释下 `checkPermissionGranted` 方法中的最后两个参数。

+ 当权限未被授予且需要向用户说明原因时，`ShowRationaleCallback` 回调函数会发出通知，在实现类的 `void needShow(final int requestCode)` 方法中自定义提示，弹出 Toast 或 对话框 等等。  
+ 布尔型的 `needRequestPermission` 参数是当权限未被授予且需要向用户说明原因时，是否直接帮你请求该权限，省去手动发起。

### 示例
请参考 [Demo](https://github.com/johnwatsondev/PhotoPicker.git) 中的 [`MainActivity`](https://github.com/johnwatsondev/PhotoPicker/blob/master/photopickerdemo/src/main/java/me/iwf/PhotoPickerDemo/MainActivity.java) 。

### 更新
开源社区已经有封装好的库，例如 [Dexter](https://github.com/Karumi/Dexter) 和 [Nammu](https://github.com/tajchert/Nammu) 。
