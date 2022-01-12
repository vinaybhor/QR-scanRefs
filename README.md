# QR-scanRefs
 Click on scan QR
        @Override
        public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
            super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == REQUEST_MULTIPLE_PERMISSIONS) {
                if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED
                        && grantResults[1] == PackageManager.PERMISSION_GRANTED && grantResults[2] == PackageManager.PERMISSION_GRANTED) {
                    // do Operation
                    Intent intent = new Intent(********.this, *******.class);
                    startActivityForResult(intent, 0);
                } else {
                    //show permission error
                }
            }
       }
       
  # On Activity Result
  
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        if (requestCode == 0) {
            if (resultCode == CommonStatusCodes.SUCCESS) {
                if (data != null) {
                    Barcode barcode = data.getParcelableExtra("barcode");
                    Constants constants = new Constants();
                    String data = "";
                    try {
                        data = barcode.displayValue;
                      
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                } else {
                    //No Barcode found
                }
            }
        } else {
            super.onActivityResult(requestCode, resultCode, data);
        }

    }
       
       -----------------------------------------------------------
 # Create Scanner screen
 # xml
 <?xml version="1.0" encoding="utf-8"?>
<RelativeLayout android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="@dimen/dp14"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="@color/colorblack">
    <FrameLayout
        android:layout_centerInParent="true"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/rounded_corner"
        >
        <SurfaceView
            android:id="@+id/surfaceView"
            android:layout_width="match_parent"
            android:layout_height="350dp"
            android:layout_gravity="center_horizontal"
            android:layout_above="@+id/btnAction"
            android:background="@drawable/border_surfaceview"
            />
    </FrameLayout>


    <TextView
        android:id="@+id/txtBarcodeValue"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_marginLeft="@dimen/activity_horizontal_margin"
        android:layout_marginTop="@dimen/activity_horizontal_margin"
        android:layout_marginStart="@dimen/activity_horizontal_margin"
        android:text="Read QR code to start Authentication"
        android:textColor="@android:color/white"
        android:textSize="20sp" />


    <Button
        android:id="@+id/btnAction"
        android:backgroundTint="@color/colorblack"
        android:textColor="@color/white"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="@dimen/activity_horizontal_margin"
        android:layout_alignParentBottom="true"
        android:text="Close" />
</RelativeLayout>


 # Activity
  @Override
    protected void onPause() {
        super.onPause();
        cameraSource.release();
    }

    @Override
    public void onResume() {
        super.onResume();
        initialiseDetectorsAndSources();
    }
    private void initViews() {
        txtBarcodeValue = findViewById(R.id.txtBarcodeValue);
        surfaceView = findViewById(R.id.surfaceView);
        btnAction = findViewById(R.id.btnAction);
        btnAction.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Constants constants = new Constants();
                constants.StoreData(getApplicationContext(), "isDestroyed", "no");
                finish();

            }
        });
    }
    private void initialiseDetectorsAndSources() {

        /*Toast.makeText(getApplicationContext(), "Barcode scanner started", Toast.LENGTH_SHORT).show();*/
        barcodeDetector = new BarcodeDetector.Builder(this)
                .setBarcodeFormats(Barcode.ALL_FORMATS)
                .build();

        cameraSource = new CameraSource.Builder(this, barcodeDetector)
                .setRequestedPreviewSize(1920, 1080)
                .setAutoFocusEnabled(true) //you should add this feature
                .build();
        surfaceView.getHolder().addCallback(new SurfaceHolder.Callback() {
            @Override
            public void surfaceCreated(SurfaceHolder holder) {
                try {
                    if (ActivityCompat.checkSelfPermission(ActivityScannedQR.this, Manifest.permission.CAMERA) == PackageManager.PERMISSION_GRANTED &&
                            ActivityCompat.checkSelfPermission(ActivityScannedQR.this, Manifest.permission.WRITE_EXTERNAL_STORAGE) == PackageManager.PERMISSION_GRANTED &&
                            ActivityCompat.checkSelfPermission(ActivityScannedQR.this, Manifest.permission.RECORD_AUDIO)==PackageManager.PERMISSION_GRANTED) {
                        cameraSource.start(surfaceView.getHolder());
                    } else {
                        ActivityCompat.requestPermissions(ActivityScannedQR.this, new
                                String[]{Manifest.permission.CAMERA, Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.RECORD_AUDIO}, REQUEST_MULTIPLE_PERMISSIONS);
                    }

                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            @Override
            public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
            }

            @Override
            public void surfaceDestroyed(SurfaceHolder holder) {
                cameraSource.stop();
            }
        });
        barcodeDetector.setProcessor(new Detector.Processor<Barcode>() {
            @Override
            public void release() {
                /*Toast.makeText(getApplicationContext(), "To prevent memory leaks barcode scanner has been stopped", Toast.LENGTH_SHORT).show();*/
            }

            @Override
            public void receiveDetections(Detector.Detections<Barcode> detections) {
                final SparseArray<Barcode> barcodes = detections.getDetectedItems();
                if (barcodes.size() != 0) {
                    Intent intent = new Intent();
                    intent.putExtra("barcode", barcodes.valueAt(0));
                    setResult(CommonStatusCodes.SUCCESS, intent);
                    finish();
                }
            }
        });
    }
    
    
    
