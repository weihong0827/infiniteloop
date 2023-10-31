---
{"Tags":["Android","Java"],"Published":"2022-07-26","Last Updated":"2022-07-26T23:30","Author":"Qiu Weihong","Created":"2022-07-26T22:38","Description":"Android App version of SUTDfolio coded using Java","publish":true,"dg-publish":true,"permalink":"/projects/sut-dfolio-android-app/sut-dfolio-android-app/","dgPassFrontmatter":true,"created":"2023-09-07T23:34:00.891+08:00","updated":"2023-10-30T20:54:21.356+08:00"}
---

# Background

Projects in SUTD that stem from course content, UROPs or ideathons are often `undocumented` despite having an `excellent` project idea or implementation. This is due to the `lack` of a `common platform` that enables students to `upload` projects and `keep track` of their `learning` throughout the school terms. Sourcing for brand new project ideas is also difficult as students do not know about the work of previous cohorts or research groups, which may also lead to `accidental plagiarism`.

As such, SUTDFolio was conceptualized to ensure `all projects` have a chance of being `documented` and `recognized` as deserved. SUTDFolio is the one-stop center of exhibition for all projects that originate from SUTD.

Ranging from `innovative undergraduate projects` to `cutting edge research theses`, SUTDFolio aims to make `every brainchild` of the SUTD community `visible to a wider audience`. Projects can be found on the project feed with a short description with more details available on click, enabling `potential employers` to `contact` the team via the attached `LinkedIn` profiles or `email` addresses. SUTD students can register and log in using their school email address to upload their projects and add descriptors via the project upload page.

# System Design and Implementation

## System architecture

![Projects/SUTDfolio Android App/attachments/Untitled.png|Untitled.png](/img/user/Projects/SUTDfolio%20Android%20App/attachments/Untitled.png)

## **App screenshots**

![Projects/SUTDfolio Android App/attachments/Untitled 1.png|Untitled 1.png](/img/user/Projects/SUTDfolio%20Android%20App/attachments/Untitled%201.png)

Fig. 1: Homepage

![Projects/SUTDfolio Android App/attachments/Untitled 2.png|Untitled 2.png](/img/user/Projects/SUTDfolio%20Android%20App/attachments/Untitled%202.png)

Fig. 2: Search Function

![Projects/SUTDfolio Android App/attachments/Untitled 3.png|Untitled 3.png](/img/user/Projects/SUTDfolio%20Android%20App/attachments/Untitled%203.png)

Fig. 3: Login and Registration

![Projects/SUTDfolio Android App/attachments/Untitled 4.png|Untitled 4.png](/img/user/Projects/SUTDfolio%20Android%20App/attachments/Untitled%204.png)

Fig. 4: Upload

  

# **Design patterns**

## **Singleton**

The singleton design pattern ensures that there is `only one instance` of an object available to a number of other classes. In our application, there is only one instance of `_SharedPreferences_` whereby all data stored in `_SharedPreferences_` can be accessed via their respective getters and new data can be updated using key value pairs. This is useful for storing the user’s _jwt token_, which is obtained only upon successful login and can be used to get the user’s information using API requests.

In the below code, the token is put into the _SharedPreferences_ instance using the _Editor_ class which can then be accessed afterwards for navigation and information retrieval. The API Request class also uses singleton design to make sure that there is only one instance of `API Request` and `_NetworkManager_` instantiated since `all the API requests` will be made using a single Android volley library `queue`

```java
APIRequest api = APIRequest.getInstance();
api.editUser(new Listener<JSONObject>() {
    @Override
    public void getResult(JSONObject object) {
        Log.d("save changes", "pass");
        navController = Navigation.findNavController(view);
        navController.navigate(R.id.action_editProfileFragment_to_profileFragment);
        Toast.makeText(getActivity(),"Changes saved.",Toast.LENGTH_LONG).show();
    }
}, setAboutMe, setPillar, setClassOf, setAvatar, jwt, view);
```

## **Adapter**

The adapter design pattern was used in `_RecyclerViewAdapter_` to obtain `post information` using API requests and display it in a given context. The code below shows the `constructor` of the adapter class, which shows the information it requires to display posts in a given layout found in the context.

```java
public class RecyclerViewAdapter extends RecyclerView.Adapter<RecyclerViewAdapter.ViewHolder>{
    private static final String TAG = "Recycler adapter";
    private List<ReadPost> posts;
    private final Context context;
    private ArrayList<ReadPost> tempList;
    private String token;
    private APIRequest request = APIRequest.getInstance();
    public RecyclerViewAdapter(Context context,ReadPost[] posts, String token) {
        this.context = context;
        this.posts = Arrays.asList(posts);
        tempList = new ArrayList<>();
        tempList.addAll(Arrays.asList(posts));
        this.token = token;
    }
	....
}
```

# Other key technical features using concepts taught

The app uses `**implicit intent**`  to allow users to `upload multiple images` from local to the cloud storage for a better showcase of their projects and to display as their profile avatar. The implicit intent triggers the `redirection` into the `photo gallery` of the phone where users select an image in local storage to upload.

  

The app uses `**asynchronous requests**` to make sure that the app is still `usable` while the backend is `processing the request sent` and `continues the job` when the process at the backend `finishes` and returns a `response`.

  

The app uses `**shared preferences**` for `data persistence` and as a method of storage for the JWT (JSON web token). This storage allows the user to `remain logged in` to their account even after closing and restarting the app.

  

The app uses **JSON objects** to communicate with the backend database, when retrieving information. Further use of the **GSON** library to `convert the JSON object` into our `defined classes` such as User and ReadPost.

  

```java
private void getPostData (){
      pref = this.getActivity().getSharedPreferences("myPrefs", Context.MODE_PRIVATE);
      token = pref.getString("token", "");
      APIRequest request = APIRequest.getInstance();
      request.getPosts(new Listener<String>() {
          @SuppressLint("NotifyDataSetChanged")
          @Override
          public void getResult(String object) {
              final Gson gson = Util.GsonParser();
              Log.d("TAG", "getResult: "+object);
              posts = gson.fromJson(object, ReadPost[].class);


              adapter = new RecyclerViewAdapter(getActivity(), posts,token);

              mRecyclerView.setAdapter(adapter);
              adapter.notifyDataSetChanged();
          }
      },token, this.getView());
  }
```

  

The API is built using Node.js with Mongodb as the database and hosted on DigitalOcean.

  

# Conclusion

The process of building this android app is very interesting, I get then chance to apply the different object-oriented design patterns into actual use. I also get to learn the basic concept of how an android app work using java which is so different from the web development that I use to do.