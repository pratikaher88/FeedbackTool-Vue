<template>

<section id="app" class="section">

    <h1 class="title is-1">
        Feedback Tool
    </h1>

    <div class="columns">
        <div class="column">
            <form @submit.prevent="submitFeedbackForm()" >
                <div class="field" >
                    <label class="label" > <strong> Comments: </strong> <span :class="{ '--exceeded': isDisabled }"> ({{ commentCharacterCount }}/500) </span></label>
                    <div class="control">
                        <textarea id="textAreaBox" rows="6" cols="60" v-model="feedbackTextArea"/>
                        <Speech2Text @speechend="speechEnd" />
                        <!-- <Speech2Text @sentTexts="sentTexts" @speechend="speechEnd" /> -->
                              <!-- {{sentences}} -->
                    </div>
                    <input type="file" @change="onImageSelected">
                    <button class="submitbutton" @submit.prevent="submitFeedbackForm" :disabled='isDisabled'>SUBMIT</button>
                </div>
            </form>
        </div>
    </div>

    <!-- <input type="text" @sentTexts="sentTexts($event)"> -->
    
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

const browser = Bowser.getParser(window.navigator.userAgent);

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

  computed: {

    commentCharacterCount() {

      if (this.isDisabled){
        return 500 - this.feedbackTextArea.length
      }
      else{
        return this.feedbackTextArea.length;
      }

    },
    isDisabled(){
      return this.feedbackTextArea.length>500;
    }

  },

  methods: {

    // sentTexts(text){
    //   console.log(text)
    // },

    speechEnd({sentences, text}) {
      console.log('text', text)
      console.log('sentences', sentences)
      this.sentences = sentences
      this.feedbackTextArea = text
    },

    submitFeedbackForm() {
      
      console.log(this.feedbackTextArea)

      this.responseData.push({"Comment": this.feedbackTextArea})
      this.responseData.push({"SiteName": window.location.href})

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

.submitbutton {
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

.submitbutton:disabled {
  background: #dddddd;
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

.--exceeded {
  color: red;

}

</style>
