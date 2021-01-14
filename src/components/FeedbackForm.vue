<template>

<section id="app" class="section">

    <h1 class="title is-1">
        Feedback Tool
    </h1>

    <div class="columns">
        <div class="column">
            <form @submit.prevent="submitFeedbackForm()">
                <div class="field">
                    <label class="label">Comments: </label>
                    <div class="control">
                        <textarea id="textAreaBox" rows="6" cols="60" v-model="feedbackTextArea"/>
                    </div>
                    <input type="file" @change="onImageSelected">
                    <button class="button" @submit.prevent="submitFeedbackForm">SUBMIT</button>
                </div>
            </form>
        </div>
    </div>

          <Speech2Text @speechend="speechEnd"/>
          {{sentences}}
    
    <br/>
    <br/>
    <br/>

    <div v-for="(value, index) in responseData" :key="index">
      <li>{{ value }}</li>
    </div>


</section>

</template>

<script>


import AWS from "aws-sdk";
import Speech2Text from "./STT"
import Bowser from "bowser";

export default {
  name: 'FeedbackForm',

  components: {
    Speech2Text
  },

  data() {
    return {
      feedbackTextArea: '',
      selectedImage: null,
      text: '',
      newArea: '',
      sentences: null,
      responseData: []
      
    }
  },

  methods: {

    speechEnd({sentences, text}) {
      console.log('text', text)
      console.log('sentences', sentences)
      this.sentences = sentences
    },

    submitFeedbackForm() {
      
      console.log(this.feedbackTextArea)

      this.responseData.push({"Comment": this.feedbackTextArea})
      this.responseData.push({"SiteName": window.location.href})

      const browser = Bowser.getParser(window.navigator.userAgent);
      console.log(browser.getBrowser());

      this.responseData.push({"BrowserName": browser.getBrowser()})

      this.feedbackTextArea = ''
      console.log("Form submitted")

      AWS.config.update({
      region: "ap-south-1",
      credentials: new AWS.CognitoIdentityCredentials({
        IdentityPoolId: "ap-south-1:946075dc-1656-44d1-92cd-4c8aa6a2ec08"
      }),});

      let s3 = new AWS.S3();
      
      if (this.selectedImage){

        var filePath = this.selectedImage.name;
      
        var params = {
            "Body": this.selectedImage,
            "Bucket": "feedbacktoolbucket",
            "Key": filePath,
            "ACL": "public-read", /* This makes the image public, but only works if your S3 bucket allows public access */
            "ContentType": this.selectedImage.type /* This is important to handle jpg vs png etc */
        };

        s3.upload(params, (err, data) => {
            if (err) {
              console.log("Error", err);
            } if (data) {
              console.log("Upload Success", data.Location);
              this.responseData.push({"ImageLocation": data.Location})
            }
          });
      }

      },
      onImageSelected(e) {
        this.selectedImage = e.target.files[0]
        console.log(this.selectedImage)
      }


  }



}

</script>

<style>

.button {
  background-color: purple; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  margin: 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 12px;
}

.upload {
  background-color: darkgreen; /* Green */
  border: none;
  color: white;
  padding: 15px 32px;
  margin: 10px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 12px;
}

</style>
