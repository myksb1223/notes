## Send image to S3 in ClassUp

It's really simple to send image to `S3`. Because Amazon offers lots of api and samples to developers.

So, here is to write down for `ClassUp` image upload algorithm.

Guess user want to change background image in schedule. `ClassUp` should remember the image until the user change the background to other. So `ClassUp` save the image to our directory.

### Directory schema

- #### failed
  User selects a background, then it will save in `failed` directory.
  S3 uploader will find the image in `failed` directory. If uploading is succeed, the image will move to `upload` directory. `move` is remove a file in previous path.

  If user selects new background, it will also save in this directory and `ClassUp` uploads and changes the image to previous in `upload` directory.
- #### upload
  Until user changes a background, `ClassUp` finds a image from `upload` path.

If `failed` and `upload` are empty, `ClassUp` starts downloading a image from `S3`.
