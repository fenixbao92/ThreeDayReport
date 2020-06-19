4-安卓-camera-拍照
===
## 使用方式
1. 调用系统相机(或包含相机功能的应用)
    ```
    public class TestActivity extends Activity {
    
        int req_1 = 1;
        int req_2 = 2;
        @BindView(R.id.iv)
        ImageView imageView;
        String filePath = null;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.testlayout);
            bind(this);
        }
    
        //拍照方式1 返回缩略图
        @OnClick(R.id.btn1)
        public void startCamera1(){
            Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
            startActivityForResult(intent,req_1);
        }
    
        //拍照方式2 返回原图
        @OnClick(R.id.btn2)
        public void startCamera2(){
            filePath = Environment.getExternalStorageDirectory().getPath()+"/tmp.png";
            Uri uri = Uri.fromFile(new File(filePath));
            Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
            intent.putExtra(MediaStore.EXTRA_OUTPUT,uri);
            startActivityForResult(intent,req_2);
        }
    
        @Override
        protected void onActivityResult(int requestCode, int resultCode, Intent data) {
            super.onActivityResult(requestCode, resultCode, data);
            if(requestCode==req_1){
                //直接使用data是缩略图形式
                Bundle bundle = data.getExtras();
                Bitmap bm = (Bitmap)bundle.get("data");
                imageView.setImageBitmap(bm);
            }else if(requestCode==req_2){
                //通过uri使用原图
                FileInputStream fis = null;
                try {
                    fis = new FileInputStream(filePath);
                    Bitmap bm = BitmapFactory.decodeStream(fis);
                    imageView.setImageBitmap(bm);
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } finally {
                    try {
                        fis.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
    ```
    testlayout.xml:
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        xmlns:android="http://schemas.android.com/apk/res/android">
        <Button
            android:id="@+id/btn1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="btn1">
        </Button>
        <Button
            android:id="@+id/btn2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="btn2">
        </Button>
        <ImageView
            android:id="@+id/iv"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
    </LinearLayout>
    ```

    
2. 自定义相机
    * 展示预览和拍照的activity
    ```
    public class Main2Activity extends Activity implements SurfaceHolder.Callback {
    
        @BindView(R.id.sv)
        SurfaceView mSurfaceView;
    
        private Camera mCamera;
        private SurfaceHolder mSurfaceHolder;
        private Camera.PictureCallback mPictureCallback = new Camera.PictureCallback() {
            @Override
            public void onPictureTaken(byte[] data, Camera camera) {
    
                Log.e("data length",data.length+"");
                //data不是缩略图 是完整数据
                File tmpFile = new File("/sdcard/tmp.jpeg");
                try {
                    FileOutputStream fos = new FileOutputStream(tmpFile);
                    fos.write(data);
                    fos.close();
                    Intent intent = new Intent(Main2Activity.this, ResultActivity.class);
                    intent.putExtra("picPath", tmpFile.getAbsolutePath());
                    startActivity(intent);
                    Main2Activity.this.finish();
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        };
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main2);
            bind(this);
            mSurfaceHolder = mSurfaceView.getHolder();
            mSurfaceHolder.addCallback(this);
            mSurfaceView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    mCamera.autoFocus(null);
                }
            });
        }
    
        @Override
        protected void onResume() {
            super.onResume();
            if (mCamera == null) {
                mCamera = getCamera();
                if (mSurfaceHolder != null) {
                    setStartPreview(mCamera, mSurfaceHolder);
                }
            }
        }
    
        @Override
        protected void onPause() {
            super.onPause();
            releaseCamera();
        }
    
        @OnClick(R.id.btn)
        public void btnClick() {
            Camera.Parameters parameters = mCamera.getParameters();
            parameters.setPictureFormat(ImageFormat.JPEG);
            Camera.Size cameraPreviewSize = getCameraPreviewSize(parameters);
            parameters.setPreviewSize(cameraPreviewSize.width,cameraPreviewSize.height);
            parameters.setPictureSize(cameraPreviewSize.width,cameraPreviewSize.height);
    //        parameters.setPictureSize(2048,1152);
    
    //        parameters.setPreviewSize(800, 400);
            parameters.setFocusMode(Camera.Parameters.FLASH_MODE_AUTO);
            mCamera.setParameters(parameters);
            mCamera.autoFocus(new Camera.AutoFocusCallback() {
                @Override
                public void onAutoFocus(boolean success, Camera camera) {
                    if (success) {
                        mCamera.takePicture(null,null,mPictureCallback);
                    }
                }
            });
    
        }
    
        //获取Camera对象
        private Camera getCamera() {
            Camera camera;
            try {
                camera = Camera.open();
            } catch (Exception e) {
                camera = null;
                e.printStackTrace();
            }
            return camera;
        }
    
        private void setStartPreview(Camera camera, SurfaceHolder surfaceHolder) {
            try {
                camera.setPreviewDisplay(surfaceHolder);
                //将系统的默认camera预览角度进行调整
                camera.setDisplayOrientation(90);
                camera.startPreview();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
        //释放相机资源
        private void releaseCamera() {
            if (mCamera != null) {
                mCamera.setPreviewCallback(null);
                mCamera.stopPreview();
                mCamera.release();
                mCamera = null;
            }
        }
    
        @Override
        public void surfaceCreated(SurfaceHolder holder) {
            setStartPreview(mCamera, mSurfaceHolder);
        }
    
        @Override
        public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
            mCamera.stopPreview();
            setStartPreview(mCamera, mSurfaceHolder);
        }
    
        @Override
        public void surfaceDestroyed(SurfaceHolder holder) {
            releaseCamera();
        }
    
        private Camera.Size getCameraPreviewSize(Camera.Parameters parameters) {
            List<Camera.Size> list = parameters.getSupportedPreviewSizes();
            Camera.Size needSize = null;
            for (Camera.Size size : list) {
                Log.e("x preview ","h"+size.height+" w"+size.width);
                if (needSize == null) {
                    needSize = size;
                    continue;
                }
                if (size.width >= needSize.width) {
                    if (size.height > needSize.height) {
                        needSize = size;
                    }
                }
            }
            return needSize;
        }
    }
    
    xml:
    <LinearLayout android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <Button
        android:id="@+id/btn"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="btn">
    </Button>
    <SurfaceView
        android:id="@+id/sv"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
    </LinearLayout>
    ```

    * 展示拍照结果的activity
    ```
    public class ResultActivity extends Activity {
    
        @BindView(R.id.iv)
        ImageView mImageView;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_result);
            bind(this);
            String path = getIntent().getStringExtra("picPath");
            try {
                //将bitmap旋转90度
                FileInputStream fis = new FileInputStream(path);
                Bitmap bitmap = BitmapFactory.decodeStream(fis);
                Matrix matrix = new Matrix();
                matrix.setRotate(90);
                bitmap = Bitmap.createBitmap(bitmap,0,0,bitmap.getWidth(),bitmap.getHeight(),matrix
                ,true);
                mImageView.setImageBitmap(bitmap);
            } catch (FileNotFoundException e) {
                e.printStackTrace();
            }
        }
    }
    xml:
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <ImageView
            android:id="@+id/iv"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:scaleType="center">
        </ImageView>
    </LinearLayout>

    ```
