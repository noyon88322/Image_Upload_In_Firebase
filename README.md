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

    //Image Piker
    implementation 'com.github.dhaval2404:imagepicker:2.1'

    //Glide Library
    implementation 'com.github.bumptech.glide:glide:4.16.0'


   ```

   ## Step:4
 in Project Setting

   ```bash

allprojects {
   repositories {
       	maven { url "https://jitpack.io" }
   }
}

    

   ```

   ## Step : 5
   I try it fragment.

 ```bash
     ImageView profile_IV;

    ActivityResultLauncher<Intent> imagePikerLauncher;

    Uri selectedImageUri;
    
    public profileFragment() {

    }


    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        imagePikerLauncher = registerForActivityResult(new ActivityResultContracts.StartActivityForResult(),
                result ->{

            if (result.getResultCode() == Activity.RESULT_OK){

                Intent data = result.getData();
                if (data!=null && data.getData()!=null){
                    selectedImageUri = data.getData();
                    Android_Util.setProfilePic(getContext(),selectedImageUri,profile_IV);

                }
            }
                }

        );
    }

            profile_IV.setOnClickListener(v -> {
            ImagePicker.with(this).cropSquare().compress(512).maxResultSize(512,512)
                    .createIntent(new Function1<Intent, Unit>() {
                        @Override
                        public Unit invoke(Intent intent) {
                            imagePikerLauncher.launch(intent);
                            return null;
                        }
                    });
        });

        profile_update_btn.setOnClickListener(v -> {

            updatebtnClick();

        });

```



## Step: 6

Make Void For Update button



   ```bash
   
     void updatebtnClick(){

        String newUserName = profile_username.getText().toString();
        if(newUserName.isEmpty() || newUserName.length()<3){
            profile_username.setError("Username Shoud Be at Least 3 Character");
            return;
        }
        currentUserModel.setUsername(newUserName);
        setInProgess(true);

        if (selectedImageUri!=null){
            FirebaseUtil.getCurrentProfilePicStorageRef().putFile(selectedImageUri)
                    .addOnCompleteListener(task -> {
                        updateToFirebase();
                    });
        }else {
            updateToFirebase();
        }

        updateToFirebase();


    }


   ```







## Step: 7

*Make a  class



   ```bash
    public class Android_Util {

    public static void setProfilePic(Context context, Uri imageUri, ImageView imageView){

        Glide.with(context).load(imageUri).apply(RequestOptions.circleCropTransform()).into(imageView);

    }


 }
 


   ```

## Step: 8

*Make a  class



   ```bash
    public class FirebaseUtil {

    public static StorageReference getCurrentProfilePicStorageRef(){
        return FirebaseStorage.getInstance().getReference().child("profilepic")
                .child(FirebaseUtil.currentUserId());
    }


 }
 


   ```
