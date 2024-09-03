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



package com.example.admin.Activity;

import android.Manifest;
import android.app.ProgressDialog;
import android.content.Intent;
import android.graphics.Color;
import android.graphics.drawable.ColorDrawable;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.appcompat.widget.AppCompatButton;
import androidx.recyclerview.widget.GridLayoutManager;
import androidx.recyclerview.widget.RecyclerView;

import com.example.admin.Adapter.Bg_photo_Adapter;
import com.example.admin.R;
import com.example.admin.model.PhotoModel;
import com.google.android.gms.tasks.OnCompleteListener;
import com.google.android.gms.tasks.OnFailureListener;
import com.google.android.gms.tasks.OnSuccessListener;
import com.google.android.gms.tasks.Task;
import com.google.android.material.floatingactionbutton.FloatingActionButton;
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

import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.List;
import java.util.Locale;

import de.hdodenhof.circleimageview.CircleImageView;

public class Bg_PhotoActivity extends AppCompatActivity {

    ProgressDialog dialog;
   RecyclerView recylerView;
   FloatingActionButton addPhoto;

    private Uri photoUri;
    private String imageUrl;
    String currentDateAndTime;
    private DatabaseReference catRef;
    private static final int IMAGE_PIKER = 1;


    ArrayList<PhotoModel>list;
    Bg_photo_Adapter adapter;

    CircleImageView upload_photo;



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_bg_photo);

        ///Progress
        dialog = new ProgressDialog(this);
        dialog.setTitle(R.string.app_name);
//        dialog.setMessage("Please Wait....");
        dialog.setCancelable(false);
        catRef = FirebaseDatabase.getInstance().getReference();



        addPhoto = findViewById(R.id.addPhoto);
        recylerView = findViewById(R.id.recylerView);

        GridLayoutManager gridLayoutManager = new GridLayoutManager(getApplicationContext(),2);
        recylerView.setLayoutManager(gridLayoutManager);



        list = new ArrayList<>();
        adapter = new Bg_photo_Adapter(list,getApplicationContext());
        recylerView.setAdapter(adapter);
        loadData();

        addPhoto.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                ShowDialogBox();
            }
        });

    }///End On create ================================




    private void ShowDialogBox (){
        final AlertDialog.Builder alert = new AlertDialog.Builder(this);
        View view = getLayoutInflater().inflate(R.layout.add_photo_box, null);
        upload_photo = view.findViewById(R.id.upload_photo);

        AppCompatButton cancel = view.findViewById(R.id.cancel);
        AppCompatButton addphotoBtn = view.findViewById(R.id.addphotoBtn);


        //*******Clicklistener

        upload_photo.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                imagInfo();
            }
        });



        //*******Clicklistener

        alert.setView(view);

        final AlertDialog alertDialog = alert.create();
        alertDialog.setCancelable(false);
        alertDialog.getWindow().setBackgroundDrawable(new ColorDrawable(Color.TRANSPARENT));

        cancel.setOnClickListener(view1 -> {
            alertDialog.dismiss();
        });

        addphotoBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                uploadImage();

            }
        });
        alertDialog.show();
    }





    private void imagInfo(){


        Dexter.withContext(getApplicationContext())
                .withPermissions(android.Manifest.permission.WRITE_EXTERNAL_STORAGE
                        , Manifest.permission.READ_EXTERNAL_STORAGE)
                .withListener(new MultiplePermissionsListener() {
                    @Override
                    public void onPermissionsChecked(MultiplePermissionsReport multiplePermissionsReport) {

                        if (multiplePermissionsReport.areAllPermissionsGranted()) {

                            Intent intent = new Intent(Intent.ACTION_PICK);
                            intent.setType("image/*");
                            startActivityForResult(intent,IMAGE_PIKER);
                        }else {
                            Toast.makeText(getApplicationContext(), "Please Allow Permission", Toast.LENGTH_SHORT).show();
                        }
                    }


                    @Override
                    public void onPermissionRationaleShouldBeShown(List<PermissionRequest> list, PermissionToken permissionToken) {

                    }
                }).check();
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        if (requestCode == IMAGE_PIKER && resultCode == RESULT_OK){

            if (data != null){

                photoUri = data.getData();
                upload_photo.setImageURI(photoUri);

            }

        }

    }


    private  void  uploadImage(){
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault());
        currentDateAndTime = sdf.format(new Date());
        if (photoUri == null){
            return;
        }
        String fileName = currentDateAndTime+".jpg";


        FirebaseStorage storage = FirebaseStorage.getInstance();
        final StorageReference storageReference = storage.getReference().child("Imageback/"+fileName);

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
                        Toast.makeText(getApplicationContext(), "Error: "+e.getMessage(), Toast.LENGTH_SHORT).show();

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

        HashMap<String, Object> hashMap =new HashMap<>();
        hashMap.put("image",imageUrl);

        String UNIQ = catRef.push().getKey();


        catRef.child("Background_photo").child(UNIQ)
                .updateChildren(hashMap)
                .addOnCompleteListener(new OnCompleteListener<Void>() {
                    @Override
                    public void onComplete(@NonNull Task<Void> task) {

                        dialog.dismiss();
                        Toast.makeText(getApplicationContext(), "Image Update Succesfully", Toast.LENGTH_SHORT).show();
                    }
                });


    }


    private void loadData()
    {
        catRef.child("Background_photo").addValueEventListener(new ValueEventListener() {
            @Override
            public void onDataChange(@NonNull DataSnapshot snapshot) {

                    for (DataSnapshot dataSnapshot : snapshot.getChildren()) {
                        PhotoModel model = dataSnapshot.getValue(PhotoModel.class);
                        list.add(model);
                        Toast.makeText(getApplicationContext(), "Success", Toast.LENGTH_SHORT).show();
                    }
                    adapter.notifyDataSetChanged();
            }

            @Override
            public void onCancelled(@NonNull DatabaseError error) {
                Toast.makeText(getApplicationContext(), "Error"+error.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });



    }





}




```

