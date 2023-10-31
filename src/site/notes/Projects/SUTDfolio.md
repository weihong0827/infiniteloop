---
{"tags":["DigitalOcean","Docker","Nodejs","Projects","Quilljs","Reactjs","Redis","DevOps"],"creation date":"2023-10-27 15:22","modification date":"Friday 27th October 2023 15:22:30","target date":null,"publish":true,"Author":"Qiu Weihong","Description":"The technical specifications behind the creation of sutdfolio","aliases":["SUTDfolio"],"dg-publish":true,"permalink":"/projects/sut-dfolio/","dgPassFrontmatter":true,"created":"2023-10-27T15:22:30.770+08:00","updated":"2023-10-31T00:10:04.418+08:00"}
---


# Intro

### Motivation

As a school well know for project oriented curriculum and focus on developing student’s design thinking mindset. Every Singapore University of Technology and Design (SUTD) student have put in a lot of effort during their school terms to create many wonderful projects that tackles many real world problems. However, many of these projects are remind unrecognised and archieved after the academic term that they suppose to work on.

### Benefits

Which we feel it is a waste and we thought of SUTDfolio to help students gain recognition of the projects they have created.

Additionally, next batches of students can use this platform as source of inspiration for their own personal/school curriculum projects.

If they find some of the projects interesting, they can even contact the seniors and maybe take over the project and work on it. In this way, some of the brightest ideas are not remain unseen and continue been worked on and improved on.

# Authorisation

As a platform for people to upload their own content and make interaction with other people’s content. A very important component to work on is authentication. This is to authorise the user actions and make sure that everyone of them is legal and will not break the system.

## Different authorisation methods

Some of the authorisation medium considered are mainly the traditional token authentication and jwt authentication.

## JSON Web Token (jwt) authentication

Jwt token are `stateless token` that contains part of the user information and can be decoded easily using various tools such as [jwt.io](https://jwt.io/).

### Advantages

- Bearer token authentication with jwt prevent [Cross-Site Request Forgery](https://owasp.org/www-community/attacks/csrf) (CSRF) attack
- Session information is not stored in the server hence save `server memory` consumption
- Data signing allows the client side users to make sure that the information is `authentic`. This ensuring the `trust` and `security` between application and it's user

### Disadvantages

- Login states are hard to manage since the token is `stateless`
- When user logout, the old token is still valid for use until it expires
- When user change password, the old token signed with the old password is still valid

## Securely use JWT

If jwt token are not managed properly, it might be exposed to malicious hackers who can use the JWT token obtained to perform user action on the user’s behalf. To use JWT properly, we need the following steps:

1. use two types of tokens: `access_token` and `refresh_token`
2. When user is authenticated using their username and password or any sort of 2FA. Server signs both the `access_token` and `refresh_token` and send it to the client side application
3. `refresh_token` typically have double the expiration time than the `access_token`. It is used to sign a new `access_token` when the `access_token` expires but `refreah_token` is still `valid`
4. `refresh_token` has to be sent via `http only cookie` so that malicious hackers cannot obtain the token via `XSS`. Http cookie will be `attached automatically` when making the subsequent http call
5. `access_token` is stored in the client memory. In this case `react state`. If that is the case we have to disable `react dev tool` from accessing our project in `production`, in case anyone can obtain the token through the react Dev tool. This can be achieved by [this package](https://www.npmjs.com/package/@fvilers/disable-react-devtools)
6. Custom `axios hooks` that will automatically request for new `access_token` using the `refresh_token` when it expires

  

## Code snippets in achieving these

### Backend

Signing two tokens when authentication using email and password and 2FA is successful

```javascript
access_token = jwt.sign({ _id: user._id, studentId: user.studentId }, process.env.ACCESS_TOKEN_SECRET, { expiresIn: '24h' })
refresh_token = jwt.sign({ _id: user._id, studentId: user.studentId }, process.env.REFRESH_TOKEN_SECRET, { expiresIn: '7d' })
```

Setting the refresh token in http only cookie

```javascript
res.cookie(`${req.origin}-jwt`, refresh_token, { httpOnly: true, maxAge: 7 * 24 * 3600 * 1000 })
```

###   
Frontend

`useRefreshToken.js` calls the refresh endpoint with the parameter `withCredentials:true` to attach the http cookie containing the refresh token. If access token is obtained successfully, it is then store in the app context state `token`

```javascript
import axios from '../axiosConfig'
import useAppContext from '../hooks/useAuth'
import {useEffect} from 'react'
const useRefreshToken = ()=>{
  const {setToken} = useAppContext()
  const refresh = async()=>{
    const response = await axios.get('/api/user/jwt/refresh',{
      withCredentials:true
    })
    setToken(prevState=>(response.data.Token))

    return response.data.Token

  }
  return refresh
  
}
export default useRefreshToken;
```

  

`useAxiosPrivate.js` is a custom hook that uses `axios interceptor` to request for a new access token using the `useRefreshToken` hook when the old request is rejected because of expire token.

After the new access token is obtained, the `axios interceptor` will then make the same request again using the new token

The code is not updated to use the bearer token scheme for authentication, but still only uses a common header `auth-token`

```javascript
import { axiosPrivate } from "../axiosConfig";
import { useEffect } from 'react';
import useRefreshToken from './useRefreshToken'
import useAppContext from '../hooks/useAuth'
const useAxiosPrivate = () => {
  const refresh = useRefreshToken();
  const { state, token } = useAppContext();
  useEffect(() => {
    const requestIntercept = axiosPrivate.interceptors.request.use(
      config => {
        if (!config.headers.common['auth-token']) {
          config.headers.common['auth-token'] = token
        }
        return config;
      }, (error) => Promise.reject(error)
    )
    const responseIntercept = axiosPrivate.interceptors.response.use(
      response => response,
      async (error) => {
        const prevRequest = error?.config;
        if (error?.reponse?.status === 403 && !prevRequest?.sent) {
          prevRequest.sent = true;
          const newAccessToken = await refresh();
          prevRequest.headers.common['auth-token'] = newAccessToken;
          return axiosPrivate(prevRequest)
        }
        return Promise.reject(error)
      }
    )

    return () => {
      axiosPrivate.interceptors.request.eject(requestIntercept)
      axiosPrivate.interceptors.response.eject(responseIntercept)
    }
  }, [token, refresh])
  return axiosPrivate
}
export default useAxiosPrivate;
```

### Future improvement

Use [Oaut2.0](https://oauth.net/2/) for more secure authorization

# Quill js

Since the sole purpose of the project is for students to upload their own projects, there must be a way for the students to elaborate their ideas for their projects and showcase it well to the public.

One of the easiest and most common way is through rich text editor. where users can perform basic actions such as `bold`, `italic`, `underline`, `code snippets`, `image attachment` and `equation`.

`Markdown` is also one of the possible alternatives to rich text editor. However, markdown may not be very user friendly since not everyone is familiar with the markdown `syntax` and constructing a more user friendly editor using markdown renders more work.

Hence I go for one of the existing library `quilljs` for richtext

I used a react wrapper library for [quill.js](https://quilljs.com/) called [react-quill](https://www.npmjs.com/package/react-quill)

  

## Code snippets

Some of the Quill js modules i used

```javascript
export const modules = {
    syntax: {
        highlight: text => hljs.highlightAuto(text).value,
    },

    toolbar: [
        ["bold", "italic", "underline", "strike"], // toggled buttons
        ["blockquote", "code-block"],

        [{ list: "ordered" }, { list: "bullet" }],
        [{ script: "sub" }, { script: "super" }], // superscript/subscript
        [{ header: [1, 2, 3, false] }],

        ["link", "image", "formula"],

        [{ color: [] }, { background: [] }], // dropdown with defaults from theme
        [{ align: [] }],

        ["clean"] // remove formatting button
    ],
    imageUploader: {
        upload: (file) => {
            return new Promise((resolve, reject) => {
                const formData = new FormData();
                formData.append("file", file);
                axios.post('/api/file?dest=contentImage&compress=true', formData)

                    .then((result) => {
                        console.log(result);
                        resolve(result.data[0].compressUrl);
                    })
                    .catch((error) => {
                        reject("Upload failed");
                        console.error("Error:", error);
                    });

            });
        }
    },
    clipboard: {
        // toggle to add extra line breaks when pasting HTML:
        matchVisual: false,
    }
}
```

For the `imageUploader` module. It uses a third party library called [quill-image-uploader](https://www.npmjs.com/package/quill-image-uploader) Which customise the behaviour on image insertion. In this case, the image will be uploaded to the backend. which will then be store in Digital Ocean space which uses AWS S3 under the hood.

  

Set up the editor

```javascript
<ReactQuill
  ref={reactQuillRef}
  defaultValue={content}
  value = {content}
  onChange={handleContentChange}
  modules={modules}
  formats={formats}
  theme={"snow"}
></ReactQuill>
```

`reactQuillRef` is initialised with react’s `useRef` hook to make sure that i can access the content inside the editor and make future customisation if i want.

# Better recommendations

## Scheduled tasks

Various scripts are written to calculate the performance of different `tags`, `courses`, and `posts`. They are run once a day automatically by the backend. With these data updated in the database, The website can give recommendation of the more popular posts, tags and courses.

## Event based recommendations

This is a feature to be worked on, where during an event where a lot of projects submission is to be done. The orgranising community can create an `event` and group all the relevant posts there. Which will then get a dedicated section on the website.

# Animation

I have also integrated some transition and interaction effects using `react-spring` for better experience. I am relatively new to animation and react-spring, hence I only used some of the template code given and tweak some of the parameters from there.

## Code snippets

  

### **Page transition**

The `position` have to be specified for the animation to behave correctly. Page transition is done to page level routes so that whenever it switch from one route to another, it will carry out the animation.

The `cofig` in the `useTransition` properties gives the animation some physical property so that the animation looks more natural.

```javascript
const transitions = useTransition(location, {
  ref: transRef,
  keys: null,
  from: { opacity: 0, scale: 0, position: 'absolute' },
  enter: { opacity: 1, scale: 1, transform: 'translate3d(0%,0,0)', position: 'relative' },
  leave: { opacity: 0, scale: 0, transform: 'translate3d(-100%,0,0)', position: 'absolute' },
  delay: 200,
  config: { mass: 1, tension: 100, friction: 20 }
  // exitBeforeEnter: true
})
useEffect(() => {
    transRef.start()
  }, [location])
return(
	
	{transitions((props, item) => (
	  <animated.div style={props} className = 'bgOffWhite'>
	    <Navbar />
	    <Routes location={item}>
	      <Route path="/" element={<Homepage />} />
	      <Route path="/projects" element={<Projects />} />
	      <Route exact path="/projects/:projectID" element={<ProjectPage />}></Route>
	      <Route path="/about" element={<About />} />
	      <Route path="/contact" element={<Contact />} />
	      <Route path="/login" element={<Login />} />
	      <Route path="/register" element={<Register />} />
	      <Route path="/upload" element={<Upload />} />
	      <Route exact path="/profile/:userId" element={<ProfilePage />} />
	      <Route exact path="/edit/profile" element={<EditProfilePage />} />
	    </Routes>
	    <Footer />
	    <CookiePolicy open={cookieOpen} setOpen={setCookieOpen} />
	  </animated.div>
	))}
)
```

### Hover animation

This gives the hovering effect on each individual post and it will rotate base on the mouse position.

```javascript
useEffect(() => {
  const preventDefault = (e) => e.preventDefault()
  document.addEventListener('gesturestart', preventDefault)
  document.addEventListener('gesturechange', preventDefault)

  setProjectData(data)
  return () => {
    document.removeEventListener('gesturestart', preventDefault)
    document.removeEventListener('gesturechange', preventDefault)
  }

}, [])

const target = useRef(null)
const [{ x, y, rotateX, rotateY, rotateZ, zoom, scale }, api] = useSpring(
  () => ({
    rotateX: 0,
    rotateY: 0,
    rotateZ: 0,
    scale: 1,
    zoom: 0,
    x: 0,
    y: 0,
    config: { mass: 5, tension: 350, friction: 40 },
  })
)


useGesture(
  {
    onMove: ({ xy: [px, py], dragging }) =>
      !dragging &&
      api({
        rotateX: calcX(py, y.get()),
        rotateY: calcY(px, x.get()),
        scale: 1.05,
      }),
    onHover: ({ hovering }) =>
      !hovering && api({ rotateX: 0, rotateY: 0, scale: 1 }),

  },
  { target, eventOptions: { passive: false } }
)
return (
	<animated.div
	ref={target}
	className={'relative rounded-xl shadow-2xl will-change-transform overflow-hidden transition-shadow duration-500 transition-opacity duration-500 hover:shadow-4xl'}
	style={{
	  transform: 'perspective(10000px)',
	  x,
	  y,
	  scale: to([scale, zoom], (s, z) => s + z),
	  rotateX,
	  rotateY,
	  rotateZ,
	}}>
		{children}
	</animated.div>
)
```

  

# Redis

Redis is used for caching and searching of posts in order to achieve better overall performance. the implementation of redis caching reduces the time taken to fetch data from an average of `100ms` to an average of `10ms`

## Setup using docker

- Pulling the image from docker hub: `docker pull redislabs/redisearch`
- Running the instance: `docker run -p 6379:6379 redislabs/redisearch:latest`
- Access the redis database using `redis-cli` and adjust the `ACL` accordingly and disable the `default` user for better security. More can be found [here](https://redis.io/docs/manual/security/acl/)

## Caching

Currently all post content are cached in the database. When there is a content update, both the content in redis and mongodb will be updated. Moving on as the the database gets bigger, `replacement policies` such as `Least Recently Used (LRU)` can be used to only cache the popular posts.

## Redis search and redis json

Since the database that we are using is mongodb, it is better to store the data in json format and it is better for content indexing too.

### Create index

The index is created using the following command

```
FT.CREATE idx:post ON JSON PREFIX 1 posts: SCHEMA $.title AS title TEXT $.desc AS desc TEXT $.courseNo._id AS course TAG $.tag.*.name AS postTag TAG $.publish AS publish TAG $.upvoteCount AS upvote NUMERIC SORTABLE $.term AS term NUMERIC $.createdTime AS time NUMERIC SORTABLE
```

Where `title` and `description` is indexed as `TEXT` for searching of the relevant content.

While content like `course` and `tags` and `publish` are indexed are `TAG` so that post of a specific content can be fetched faster.

# Deployment

## Digital Ocean managed database

For better `security` consideration, I opted to go for the Digital Ocean `managed database` for `mongodb` instead of setting up myself using a `droplet`.

I did not use Digital Ocean’s service for `redis` because they do not support `redis-serach` due to some licencing issues. Hence I have to set up one myself using a `droplet` and carry out the relevant security measures to make sure that the data is secured.

## Digital Ocean Space

Digital Ocean Space is mainly used for the storage of images. It uses AWS S3 under the hood. To connect to it, we need a s3 client.

```javascript
const { S3 } = require('@aws-sdk/client-s3')
const dotenv = require('dotenv')

dotenv.config()

const s3Client = new S3({
    endpoint: process.env.DO_SPACES_ENDPOINT,
    region: "sgp1",
    credentials: {
        accessKeyId: process.env.DO_SPACES_ACCESS_KEY,
        secretAccessKey: process.env.DO_SPACES_SECRET_ACCESS_KEY
    }
});

module.exports = s3Client
```

The documentation of how to use s3 bucket to carry out CRUD of the objects can be found [here](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-s3/index.html)

### Digital Ocean App platform

This is used to host the website backend. The experience is great in terms of setting up and integration with github to achieve continuous deployment.

  

# Moving Forward

The inital phase of development of SUTDfolio has been done. Further development is definitely needed to improve on the platform’s security, performance and interaction.

## Features

Here are some of the features that I have in mind.

- Endorsement of projects by faculty members, to increase credibility
- Embedding of 3D models into the project details page for visitors to interact with
- Improved UI for project recommendation feature
- Adding individual group member’s specific contribution to the project, rather than just a general description

## Deployment

I have recently take the `AWS Cloud Practitioner` certification (more info) and I have learnt a lot about the services that AWS has to offer. Hence I am thinking to shift the entire project to AWS since its is more well-established and has more features that we can leverage on.

# Conclusion

I really have learn a lot from this project, even tho it might seem to be a very easy project of just uploading and displaying, but there are really so much things going on behind the scene to make the app working fast, secure and user friendly. That is why I love programming, because I can really make use of it and create a lot of cool features.