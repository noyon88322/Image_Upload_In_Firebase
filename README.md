## Image Upload  in Firebase 

## Step:1

1.Go to firebase and Bulid>Storage>create

2.Select (Start in test mode) 


## Step:2
--In Studio connect

![Screenshot (25)](https://github.com/user-attachments/assets/3eae0f00-99cd-4dc6-8048-3e4b29edef0e)


## Step:3
Add gradle

   ```bash


    repositories {
        google()
        mavenCentral()
        maven { url "https://jitpack.io" }
    }


    ///Kamrumi
    implementation 'com.karumi:dexter:6.2.3'

    //Glide Library
    implementation 'com.github.bumptech.glide:glide:4.16.0'


   ```

   ## Step:4
open Manifest

   ```bash

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

    <application
    

   ```

   ## Step : 5
   I try it fragment.

 ```bash
    package com.example.watchandearn;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.AppCompatButton;

import android.Manifest;
import android.app.ProgressDialog;
import android.content.Intent;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import com.bumptech.glide.Glide;
import com.example.watchandearn.model.ProfileModel;
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.OnFailureListener;
import com.google.android.gms.tasks.OnSuccessListener;
import com.google.android.gms.tasks.Task;
import com.google.firebase.auth.FirebaseAuth;
import com.google.firebase.auth.FirebaseUser;
import com.google.firebase.database.DataSnapshot;
import com.google.firebase.database.DatabaseError;
import com.google.firebase.database.DatabaseReference;
import com.google.firebase.database.FirebaseDatabase;
import com.google.firebase.database.ValueEventListener;
import com.google.firebase.storage.FirebaseStorage;
import com.google.firebase.storage.OnProgressListener;
import com.google.firebase.storage.StorageReference;
import com.google.firebase.storage.UploadTask;
import com.karumi.dexter.Dexter;
import com.karumi.dexter.MultiplePermissionsReport;
import com.karumi.dexter.PermissionToken;
import com.karumi.dexter.listener.PermissionRequest;
import com.karumi.dexter.listener.multi.MultiplePermissionsListener;

import java.util.HashMap;
import java.util.List;

import de.hdodenhof.circleimageview.CircleImageView;

public class ProfileActivity extends AppCompatActivity {

    private CircleImageView circleImageView;
    private ImageView img_edit_profile;
    private TextView tv_name,tv_email,tv_share,tv_history,tv_logout;
    private AppCompatButton updateBtn;
    private  DatabaseReference reference;
    private FirebaseUser user;
    private  FirebaseAuth auth;

    ProgressDialog dialog;
    private static final int IMAGE_PIKER = 1;
    private Uri photoUri;
    private String imageUrl;



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_profile);

        ///Progress
        dialog = new ProgressDialog(this);
        dialog.setTitle(R.string.app_name);
//        dialog.setMessage("Please Wait....");
        dialog.setCancelable(false);

        ////Identity
        circleImageView = findViewById(R.id.circleImageView);
        img_edit_profile = findViewById(R.id.img_edit_profile);
        tv_name = findViewById(R.id.tv_name);
        tv_email = findViewById(R.id.tv_email);
        tv_share = findViewById(R.id.tv_share);
        tv_history = findViewById(R.id.tv_history);
        tv_logout = findViewById(R.id.tv_logout);
        updateBtn = findViewById(R.id.updateBtn);

        ////Fireebase

        auth = FirebaseAuth.getInstance();
        user = auth.getCurrentUser();
        reference = FirebaseDatabase.getInstance().getReference().child("users");

        ///Method Cell
        loadDataFromDatabase();




        updateBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                uploadImage();
            }
        });

        img_edit_profile.setOnClickListener(v -> {
            Dexter.withContext(ProfileActivity.this)
                    .withPermissions(Manifest.permission.WRITE_EXTERNAL_STORAGE
                            ,Manifest.permission.READ_EXTERNAL_STORAGE)
                    .withListener(new MultiplePermissionsListener() {
                        @Override
                        public void onPermissionsChecked(MultiplePermissionsReport multiplePermissionsReport) {

                            if (multiplePermissionsReport.areAllPermissionsGranted()) {

                                Intent intent = new Intent(Intent.ACTION_PICK);
                                intent.setType("image/*");
                                startActivityForResult(intent,IMAGE_PIKER);
                            }else {
                                Toast.makeText(ProfileActivity.this, "Please Allow Permission", Toast.LENGTH_SHORT).show();
                            }
                        }


                        @Override
                        public void onPermissionRationaleShouldBeShown(List<PermissionRequest> list, PermissionToken permissionToken) {

                        }
                    }).check();
        });



    }

    ////Method make
    public   void  loadDataFromDatabase(){

        reference.child(user.getUid())
                .addValueEventListener(new ValueEventListener() {
                    @Override
                    public void onDataChange(@NonNull DataSnapshot snapshot) {

                        ProfileModel model = snapshot.getValue(ProfileModel.class);

                        tv_name.setText(model.getName());
                        tv_email.setText(model.getEmail());
                        Glide.with(getApplicationContext())
                                .load(model.getImage())
                                .placeholder(R.drawable.avatar)
                                .timeout(6000)
                                .into(circleImageView);


                    }

                    @Override
                    public void onCancelled(@NonNull DatabaseError error) {
                        Toast.makeText(ProfileActivity.this, "Error: "+error.getMessage(), Toast.LENGTH_SHORT).show();
                        finish();
                    }
                });


    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == IMAGE_PIKER && resultCode == RESULT_OK){

            if (data != null){

                photoUri = data.getData();
                circleImageView.setImageURI(photoUri);
                updateBtn.setVisibility(View.VISIBLE);
            }

        }

    }


    private  void  uploadImage(){

        if (photoUri == null){
            return;
        }
        String fileName = user.getUid()+".jpg";


        FirebaseStorage storage = FirebaseStorage.getInstance();
        final StorageReference storageReference = storage.getReference().child("Image/"+fileName);

        dialog.show();

        storageReference.putFile(photoUri)
                .addOnSuccessListener(new OnSuccessListener<UploadTask.TaskSnapshot>() {
                    @Override
                    public void onSuccess(UploadTask.TaskSnapshot taskSnapshot) {

                        storageReference.getDownloadUrl()
                                .addOnSuccessListener(new OnSuccessListener<Uri>() {
                                    @Override
                                    public void onSuccess(Uri uri) {
                                        imageUrl = uri.toString();
                                        uploadImageUrlToDatabase();
                                    }


                                });

                    }
                }).addOnFailureListener(new OnFailureListener() {
                    @Override
                    public void onFailure(@NonNull Exception e) {
                        dialog.dismiss();
                        Toast.makeText(ProfileActivity.this, "Error: "+e.getMessage(), Toast.LENGTH_SHORT).show();

                    }
                }).addOnProgressListener(new OnProgressListener<UploadTask.TaskSnapshot>() {
                    @Override
                    public void onProgress(@NonNull UploadTask.TaskSnapshot snapshot) {

                        long totalS = snapshot.getTotalByteCount();
                        long transferS = snapshot.getBytesTransferred();

                        long transferSize = (totalS / 1024);
                        long totalSize = (transferS / 1024);


                        dialog.setMessage("Upload "+((int) transferSize)+"KB / "+((int) totalSize)+"KB");

                    }
                });


    }

    private void uploadImageUrlToDatabase() {

        HashMap<String, Object> map =new HashMap<>();
        map.put("image",imageUrl);

        reference.child(user.getUid())
                .updateChildren(map)
                .addOnCompleteListener(new OnCompleteListener<Void>() {
                    @Override
                    public void onComplete(@NonNull Task<Void> task) {
                        updateBtn.setVisibility(View.GONE);
                        dialog.dismiss();
                        Toast.makeText(ProfileActivity.this, "Image Update Succesfully", Toast.LENGTH_SHORT).show();
                    }
                });


    }



}

```

