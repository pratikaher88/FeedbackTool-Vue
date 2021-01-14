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

    <v-app id="inspire">
    <v-container fluid>
      <v-layout row wrap justify-center class="mt-4">
        <v-flex xs12 sm10 text-xs-center>
          <v-text-field
            label="The text"
            v-model="text"
            textarea
          ></v-text-field>
        </v-flex>
        <v-flex xs12 sm8 md4 text-xs-center>
          <Speech2Text v-model="text" @speechend="speechEnd"></Speech2Text>
        </v-flex>
        <v-flex xs12 text-xs-center class="mt-4">
          {{sentences}}
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>


</section>

</template>

<script>


import AWS from "aws-sdk";
import Speech2Text from "./STT"

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
      sentences: null
      
    }
  },

  methods: {

    speechEnd ({sentences, text}) {
      console.log('text', text)
      console.log('sentences', sentences)
      this.sentences = sentences
    },

      submitFeedbackForm() {
        this.feedbackTextArea = ''
        console.log("Form submitted")
        console.log(this.selectedImage.type)
        
        AWS.config.update({
        region: "ap-south-1",
        credentials: new AWS.CognitoIdentityCredentials({
          IdentityPoolId: "ap-south-1:946075dc-1656-44d1-92cd-4c8aa6a2ec08"
        }),

      });

        let s3 = new AWS.S3();

        // let decodedImage = Buffer.from(this.selectedImage, 'base64');
        
       
        var filePath = this.selectedImage.name;
        var params = {
            "Body": this.selectedImage,
            "Bucket": "feedbacktoolbucket",
            "Key": filePath,
            "ACL": "public-read", /* This makes the image public, but only works if your S3 bucket allows public access */
            "ContentType": this.selectedImage.type /* This is important to handle jpg vs png etc */
        };
        s3.upload(params, function (err, data) {
            console.log(err, data);
        });

        

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
