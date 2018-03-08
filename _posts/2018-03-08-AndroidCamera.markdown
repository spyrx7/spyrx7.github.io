---
layout:     post
title:      "Android Camera"
subtitle:   " \"Android Camera 简单总结\""
date:       2018-03-08 15:00:00
author:     "spyrx7"
catalog: true
tags:
    - android
---


# 使用系统相机

>模式有两种,其一,使用系统相机拍摄,不需要储存的;其二,拍摄后需要储存的.

#### 模式一  
- 向清单文件添加相机权限 
- 建立意图(Intent),并启用它
- 在onActivityResult 里获得返回的图片

##### 简单的实例

注册权限

```
   <uses-permission android:name="android.permission.CAMERA"/>
```

建立意图并启用

```

static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}

```

获取到简单的缩略图


```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
```

模式二 

- 在清单文件里添加权限
- 构建意图
- 建立图片的保存路径和名字
- 配置fileprovider (可选)
- 启动意图
- 返回onActivityResult 

> 这是获取一张全图的方法,和模式一,区别的是这个是一张完整的图,所以大小会和相机拍摄的一样,这种模式是针对对相机有高要求的开发者,步骤会比模式一复杂一些.其实也就是在意图启动之前,建立好路径和名字(名字需要保持唯一性) ,并且绑定在意图上,在拍摄完成后,我们可以通过这个路径访问这张图片.

##### 简单实例

清单文件里添加权限 (这里需要读取SD卡,所以多了访问和写入储存的权限)

```
 <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
 <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
 <uses-permission android:name="android.permission.CAMERA"/>
 
```
构建意图 ,并且建立图片的保存路径和名字,配置fileprovider(可选,仅应用自己访问的场景)

这里又分有两种,一种是公开的及保存在SD卡,其他应用程序可也以访问的,设置共享照片的正确目录由getExternalStoragePublicDirectory（）提供，并带有DIRECTORY_PICTURES参数;一种是私有的,只是应用本身访问的路径,则可以使用getExternalFilesDir（）提供的目录,在Android 4.3及更低版本上，写入此目录还需要WRITE_EXTERNAL_STORAGE权限。从Android 4.4开始，不再需要该权限，因为该目录不能被其他应用程序访问，因此您可以通过添加maxSdkVersion属性来声明仅在较低版本的Android上请求权限


下面我们来做私有方式的演示(公开的就不需要这个步骤了,不过在Android 6 及以上的需要注意动态权限的问题,当然下面的方式是不需要这个的)

在res 下 建立xml包,然后建立provider_paths.xml文件


```
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="Android/data/junjianstudio.dome/files/Pictures" />
</paths>
```


```
  <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="junjianstudio.dome.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/provider_paths" />
  </provider>


```

> note : 这里需要注意 junjianstudio.dome 都替换城你的应用的包名

建立路径和名字,保证名字的唯一性,这个使用时间戳就能完美解决了

```

    private File createImagerFile() throws IOException {
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String imagerFileName = "JPEG_" + timeStamp + "_";
        File storagDir = getExternalFilesDir(Environment.DIRECTORY_PICTURES);
        File image = File.createTempFile(imagerFileName
                ,".jpg"
                ,storagDir);

        mCurrentPhotoPath = image.getPath();

        return image;
 }


```

建立意图


```
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
         if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
                //startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
                File photoFile = null;
                try {
                    photoFile = createImagerFile();
                } catch (IOException e) {

                }

            if(photoFile != null){
                Uri photoUri = FileProvider.getUriForFile(CameraActivity.this
                        ,"junjianstudio.dome.fileprovider"
                        ,photoFile);
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, photoUri);
                startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
            }
        }


```

在onActivityResult ,通过之前保存的路径,获得图片

```
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if(requestCode == REQUEST_TAKE_PHOTO && resultCode == RESULT_OK){
            
            File f = new File(mCurrentPhotoPath);
            Log.e("CameraActivity", "onActivityResult: mCurrentPhotoPath = " + mCurrentPhotoPath);
        }
    }
```
```