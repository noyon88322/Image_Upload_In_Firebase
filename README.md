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


```
## Step: 6

*Make a class

   ```bash
    public class Android_Util {

    public static void setProfilePic(Context context, Uri imageUri, ImageView imageView){

        Glide.with(context).load(imageUri).apply(RequestOptions.circleCropTransform()).into(imageView);

    }


 }
 


   ```


