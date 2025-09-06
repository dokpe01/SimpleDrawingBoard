package com.cookandroid.mydraw;

import android.Manifest;
import android.annotation.SuppressLint;
import android.content.pm.PackageManager;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.os.Build;
import android.os.Bundle;

import androidx.annotation.RequiresApi;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import android.os.Environment;
import android.util.Log;
import android.view.View;
import android.widget.ImageButton;
import android.widget.LinearLayout;
import android.widget.Toast;

import com.cookandroid.mydraw.R;

import java.io.File;
import java.io.FileOutputStream;


public class MainActivity extends AppCompatActivity {
    private static final int MY_PERMISSIONS_REQUEST_WRITE_EXTERNAL_STORAGE = 1024;
    private com.cookandroid.mydraw.SingleTouchView drawView;
    private ImageButton currPaint;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        drawView = (com.cookandroid.mydraw.SingleTouchView) findViewById(R.id.drawing);
        LinearLayout paintLayout = (LinearLayout) findViewById(R.id.paint_colors);
        currPaint = (ImageButton) paintLayout.getChildAt(0);
        View new_btn = findViewById(R.id.new_btn);
        View draw_btn = findViewById(R.id.draw_btn);
        View erase_btn = findViewById(R.id.erase_btn);
        View save_btn = findViewById(R.id.save_btn);
        View load_btn = findViewById(R.id.load_btn);

        int permissionCheck = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
        if (permissionCheck!= PackageManager.PERMISSION_GRANTED){
            Toast.makeText(this, "권한 승인이 필요합니다.", Toast.LENGTH_SHORT).show();
            if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.WRITE_EXTERNAL_STORAGE)){
                Toast.makeText(this, "내부저장소 쓰기 권한이 필요합니다.", Toast.LENGTH_LONG).show();
            } else {
                ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, MY_PERMISSIONS_REQUEST_WRITE_EXTERNAL_STORAGE);
            }
        }

        ImageButton.OnClickListener onClickListener = new ImageButton.OnClickListener() {
            @SuppressLint({"WrongConstant", "ShowToast"})
            @RequiresApi(api = Build.VERSION_CODES.JELLY_BEAN)
            @Override
            public void onClick(View view) {
                switch (view.getId()) {
                    case R.id.load_btn:
                        paintLayout.setVisibility(View.INVISIBLE);
                        loadPicture();
                        break;
                    //새 파일
                    case R.id.new_btn:
                        drawView.setMenu(0);
                        paintLayout.setVisibility(View.INVISIBLE);
                        break;

                    //그리기 모드
                    case R.id.draw_btn:
                        drawView.setMenu(1);

                        //그리기 모드 버튼을 눌러야 색상 버튼들이 보임
                        paintLayout.setVisibility(View.VISIBLE);
                        break;

                    //지우개 모드
                    case R.id.erase_btn:
                        drawView.setMenu(2);
                        paintLayout.setVisibility(View.INVISIBLE);
                        break;

                    //저장 모드
                    case R.id.save_btn:
                        paintLayout.setVisibility(View.INVISIBLE);
                        savePicture();
                        break;

                    default:
                        break;

                }
            }
        };
        new_btn.setOnClickListener(onClickListener);
        draw_btn.setOnClickListener(onClickListener);
        erase_btn.setOnClickListener(onClickListener);
        save_btn.setOnClickListener(onClickListener);
        load_btn.setOnClickListener(onClickListener);
    }

    public void clicked(View view) {
        if (view != currPaint) {
            String color = view.getTag().toString();
            drawView.setColor(color);
        }
        else {
            drawView.setColor("#00000000");
        }
        currPaint = (ImageButton) view;
    }
    private void savePicture() {

        drawView.setDrawingCacheEnabled(true);
        Bitmap SingleTouchView = Bitmap.createBitmap(drawView.getDrawingCache());
        drawView.setDrawingCacheEnabled(false);
        File dir = Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_PICTURES);

        if(!dir.exists())
            dir.mkdirs();
        FileOutputStream outF;
        try {
            outF = new FileOutputStream(new File(dir, "image.png"));
            SingleTouchView.compress(Bitmap.CompressFormat.PNG, 100, outF);
            outF.close();
            Toast.makeText(this, "저장 성공", Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            Log.e("phoro","그림저장오류",e);
            Toast.makeText(this, "저장 실패", Toast.LENGTH_SHORT).show();
        }

    }

    private void loadPicture() {
        try {
            String imgpath = "mnt/sdcard/Pictures/image.png";
            Bitmap bitmap = BitmapFactory.decodeFile(imgpath);
            drawView.setDrawingCacheEnabled(true);
            Toast.makeText(this, "불러오기 성공", Toast.LENGTH_SHORT).show();
        } catch (Exception e) {
            Toast.makeText(this, "불러오기 실패", Toast.LENGTH_SHORT).show();
        }
    }
}