##### 1. 简介

Android 6.0 之前我们申请权限直接在配置文件中配置一下即可，但是6.0之后，谷歌官方将权限分为普通权限和危险权限。对于危险权限来说，我们就需要进行动态设置了。

[[Android官网]Permission整体介绍](https://developer.android.com/guide/topics/permissions/overview)

[[Android官网]Permission动态请求文档](https://developer.android.com/training/permissions/requesting)

[[中文]系统权限](https://developer.android.com/guide/topics/security/permissions.html?hl=zh-cn)

##### 2. 相关方法

- **ContextCompat.checkSelfPermission**
   检查应用是否具有某个危险权限。如果应用具有此权限，方法将返回 PackageManager.PERMISSION_GRANTED，并且应用可以继续操作。如果应用不具有此权限，方法将返回 PackageManager.PERMISSION_DENIED，且应用必须明确向用户要求权限。

- **ActivityCompat.requestPermissions**
   应用可以通过这个方法动态申请权限，调用后会弹出一个对话框提示用户授权所申请的权限。

- **ActivityCompat.shouldShowRequestPermissionRationale**
   如果应用之前请求过此权限但用户拒绝了请求，此方法将返回 true。如果用户在过去拒绝了权限请求，并在权限请求系统对话框中选择了 Don't ask again 选项，此方法将返回 false。如果设备规范禁止应用具有该权限，此方法也会返回 false。

- **onRequestPermissionsResult**
   当应用请求权限时，系统将向用户显示一个对话框。当用户响应时，系统将调用应用的请求权限 的Activity的onRequestPermissionsResult() 方法，向其传递用户响应，处理对应的场景。

##### 3. 危险权限

对于危险权限我们需要动态设置，不然应用会崩溃的，首先我们要清楚哪些是危险权限？
![PermissionImage](https://img-blog.csdn.net/20180408165912762?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0p1c3Rpbk5pY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

在 Android 8.0 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用。

对于针对 Android 8.0 的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而，一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准。

例如，假设某个应用在其清单中列出 `READ_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE`。应用请求 `READ_EXTERNAL_STORAGE`，并且用户授予了该权限。如果该应用针对的是 API 级别 24 或更低级别，系统还会同时授予 `WRITE_EXTERNAL_STORAGE`，因为该权限也属于同一 `STORAGE` 权限组并且也在清单中注册过。如果该应用针对的是 Android 8.0，则系统此时仅会授予 `READ_EXTERNAL_STORAGE`；不过，如果该应用后来又请求 `WRITE_EXTERNAL_STORAGE`，则系统会立即授予该权限，而不会提示用户。

### 媒体

##### 4. 请求权限框架实现

该框架只能在Activity中请求权限，如果在Fragment中，则需要getActivity()再请求权限；

使用步骤：

- 在请求权限的Activity中，需要重写onRequestPermissionsResult方法来将其事件传入PermissionHelper处理；

  ```kotlin
      override fun onRequestPermissionsResult(
          requestCode: Int,
          permissions: Array<out String>,
          grantResults: IntArray
      ) {
          super.onRequestPermissionsResult(requestCode, permissions, grantResults)
          //添加这句即可
          PermissionHelper.onRequestPermissionResult(this, requestCode, permissions, grantResults)
      }
  ```

  

- 在需要请求权限的地方调用PermissionHelper.requestPermission()方法，在其PermissionCallback接口中处理相应权限申请结果即可：

  ```kotlin
   PermissionHelper.requestPermission(
       this, //activity object
       arrayListOf("android.permission.READ_PHONE_STATE", "android.permission.READ_CALENDAR"),
       object : PermissionHelper.PermissionCallback {
           override fun onGranted() {
               Log.d(TAG, "onGranted")
           }
             override fun onDenied() {
               Log.d(TAG, "onDenied")
           }
             override fun onDeniedForever(permissions: ArrayList<String>) {
               Log.d(TAG, "onDeniedForever: $permissions")
               PermissionHelper.showPermissionAlert(activity, permissions, "BB")
           }
       })
  ```

  PermissionCallback的用法请参考PermissionCallback的接口方法说明；



实现代码：

```kotlin
/**
 * 动态申请系统权限的工具类
 */
object PermissionHelper {
    private const val REQUEST_CODE_RANGE_SIZE = 1000
    private const val PERMISSION_QUERIED_KEY_PREFIX = "HasRequestedAndroidPermission::"

    private var mNextRequestCode = 100
    private val mPermissionRequestMap by lazy { SparseArray<PermissionCallback>() }
    private val mDeniedForeverPermissionMap by lazy { SparseArray<List<String>>() }

    /**
     *
     */
    interface PermissionCallback {
        /**
         * 当所有权限用户都允许时回调, 该方法；
         */
        fun onGranted()

        /**
         * 申请的权限中有一项或以上被用户拒绝时，回调该方法；
         */
        fun onDenied()

        /**
         * 当末允许的权限，用户在过去都钩选“不再提醒”选项时，会回调该方法；
         */
        fun onDeniedForever(permissions: ArrayList<String>)
    }

    /**
     * 处理用户允许相关权限的情况，并回调给请求权限申请者
     */
    fun onRequestPermissionResult(
        activity: Activity,
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        //save requested key for check in next time.
        var editor = PreferenceManager.getDefaultSharedPreferences(activity).edit()
        permissions.forEach {
            editor.putBoolean(getRequestPermissionGroupKey(activity, it), true)
        }
        editor.apply()

        val deniedPermissions = arrayListOf<String>()
        grantResults.forEachIndexed { index, value ->
            if (value != PackageManager.PERMISSION_GRANTED) {
                deniedPermissions.add(permissions[index])
            }
        }

        if (deniedPermissions.size > 0) {
            if (mDeniedForeverPermissionMap.get(requestCode).size == deniedPermissions.size ) {
                mPermissionRequestMap.get(requestCode).onDeniedForever(deniedPermissions)
            } else {
                mPermissionRequestMap.get(requestCode).onDenied()
            }

        } else {
            mPermissionRequestMap.get(requestCode).onGranted()
        }

        mPermissionRequestMap.delete(requestCode)
    }

    /**
     * 请求权限
     */
    fun requestPermission(
        activity: Activity,
        permissions: List<String>,
        callback: PermissionCallback
    ) {
        //低于Android6.0的，不需要动态申请权限,直接返回
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            callback.onGranted()
            return
        }

        // Check whether this app have been grant all request permission.
        //val grantedPermissions = arrayListOf<String>()
        val grantedPermissions = permissions.filter {
            activity.checkSelfPermission(it) == PackageManager.PERMISSION_GRANTED
        }
        if (grantedPermissions.size == permissions.size) {
            callback.onGranted()
            return
        }

        // Collect permission which have been denied forever.
        val deniedForeverPermissions = permissions.filter {
            !canRequestPermission(activity, it) && !grantedPermissions.contains(it)
        }


        val requestCode = getNextRequestCode()
        mPermissionRequestMap.put(requestCode, callback)
        mDeniedForeverPermissionMap.put(requestCode, deniedForeverPermissions)
        activity.requestPermissions(permissions.toTypedArray(), requestCode)
    }

    /**
     * 检查该权限是否能申请;
     * 有可能系统禁止申请，有可能是用户已经拒绝并选择了“不再提醒”选项;
     */
    private fun canRequestPermission(activity: Activity, permission: String): Boolean {
        //仅针对本Permission请求框架
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) return true

        //有些权限可能会由厂商的不同策略而禁止使用
        if (activity.packageManager.isPermissionRevokedByPolicy(permission, activity.packageName)) {
            return false
        }

        if (activity.shouldShowRequestPermissionRationale(permission)) {
            return true
        }

        // Check whether we have ever asked for this permission by checking whether we saved
        // a preference associated with it before.
        return !PreferenceManager.getDefaultSharedPreferences(activity).getBoolean(
            getRequestPermissionGroupKey(activity, permission), false
        )

    }

    private fun getNextRequestCode(): Int {
        mNextRequestCode = (mNextRequestCode + 1) % REQUEST_CODE_RANGE_SIZE
        return mNextRequestCode
    }

    /**
     * get group name for request permission.
     */
    private fun getRequestPermissionGroupKey(activity: Activity, permission: String) : String {
        return PERMISSION_QUERIED_KEY_PREFIX + try {
            val permissionInfo = activity.application.packageManager.getPermissionInfo(
                permission,
                PackageManager.GET_META_DATA
            )
            if (!TextUtils.isEmpty(permissionInfo.group)) {
                permissionInfo.group
            } else {
                permission
            }
        } catch (e: PackageManager.NameNotFoundException) {
            permission
        }
    }

    /**
     * 当用户之前已经拒绝过该权限并钩选“不再提醒”选项时， 可调用该方法来显示AlertDialog 来引导用户去设置里打开权限；
     * 一般在{@link PermissionHelper.onDeniedForever}方法中调用；
     */
    fun showPermissionAlert(activity: Activity, permissions: List<String>, feature: String) {
        if (permissions.isEmpty()) return

        val inflater = activity.getSystemService(Context.LAYOUT_INFLATER_SERVICE) as LayoutInflater
        val view = inflater.inflate(R.layout.permission_alert_dialog_layout, null)

        view.findViewById<TextView>(R.id.alert_text).text =
            getPermissionSpannableString(activity, feature)

        view.findViewById<ListView>(R.id.permission_list).adapter =
            ArrayAdapter<String>(
                activity,
                android.R.layout.simple_expandable_list_item_1,
                permissions
            )

        //Show a alert dialog
        AlertDialog.Builder(activity).setNegativeButton(R.string.cancel) { dialog, _ ->
            dialog.dismiss()
        }.setPositiveButton(R.string.setting) { dialog, _ ->
            val intent = Intent().setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
                .setData(
                    Uri.fromParts(
                        "package", activity.applicationInfo.packageName, null
                    )
                )
            activity.startActivity(intent)
            dialog.dismiss()
        }.setCancelable(false).setView(view).create().show()
    }

    private fun getPermissionSpannableString(
        context: Context,
        feature: String
    ): SpannableStringBuilder {
        val permissionString = context.resources.getString(R.string.go_to_setting_for_perm, feature)
        val featureIndex = permissionString.indexOf(feature)

        val spannableStringBuilder = SpannableStringBuilder(permissionString)
        spannableStringBuilder.setSpan(
            StyleSpan(Typeface.BOLD),
            featureIndex,
            featureIndex + feature.length,
            Spanned.SPAN_EXCLUSIVE_EXCLUSIVE
        )
        return spannableStringBuilder
    }
}
```

permission_alert_dialog_layout的布局文件如下：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingTop="24dp"
    android:paddingRight="24dp"
    android:paddingLeft="24dp"
    >

    <TextView
        android:id="@+id/alert_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="start"
        android:textColor="#252525"
        android:textSize="17dp" />

    <ListView
        android:id="@+id/permission_list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="8dp"
        android:dividerHeight="3dp"
        android:divider="@android:color/transparent"
        android:layout_marginBottom="24dp" />
</LinearLayout>
```

该代码还有需要完美的地方，比如直接显示权限字符串给用户，用户理解不了之类，后面使用再更新；