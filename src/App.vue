<script>
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT license.
import * as sdk from "microsoft-cognitiveservices-speech-sdk";
import pinyin from "pinyin";

export default{
  data(){
    return{
      inputText: "",
      apiKey:"",
      voiceName:"zh-CN-XiaoxiaoNeural",
      out:{
        text:"",
        audio:null,
      },
      out_prev:{
        text:"",
        audio:null,
      },
      cfg:{
        speed:0.8,
        pitch:0.0
      }
    }
  },
  methods:{
    convert(){
      console.log(this.inputText);
      if (this.inputText.length>20 || this.inputText.length<1){
          alert("文章は1文字以上20文字以内でなければなりません。");
          return;
      }

      this.out_prev.text=this.out.text
      this.out.text=this.inputText;
    },
    playSound(buffer){
      const buffer_copy = buffer.slice(0);
      const context = new (window.AudioContext || window.webkitAudioContext)();
      let source = context.createBufferSource();
      context.decodeAudioData(buffer_copy,(buf)=>{
        source.buffer = buf;
        source.connect(context.destination);
        source.start(0);
      });
    },
    tts(){
        // <code>
        const input = this.out.text

        console.log(input);
        if (input.length>20 || input.length<1){
          alert("文章は1文字以上20文字以内でなければなりません。")
          return;
        }else if(input===this.out_prev.text){
          console.log("バッファから読み上げ実行",this.out.audio);
          this.playSound(this.out.audio);
          return;
        }
        console.log("読み上げ実行：",input)

        this.out_prev.text=input

        
        // replace with your own subscription key,
        // service region (e.g., "westus"), and
        // the name of the file you save the synthesized audio.
        const subscriptionKey = this.apiKey;
        const serviceRegion = "japaneast"; // e.g., "westus"
      
        // we are done with the setup
      
        // now create the audio-config pointing to our stream and
        // the speech config specifying the language.
        let speechConfig = sdk.SpeechConfig.fromSubscription(subscriptionKey, serviceRegion);
        speechConfig.speechSynthesisLanguage = "zh-CN";
        speechConfig.speechSynthesisVoiceName = this.voiceName;
        
        // create the speech synthesizer.
        const synthesizer = new sdk.SpeechSynthesizer(speechConfig);

        const mode='xssml';
        if(mode==='ssml'){
          let ssml=``
          if(this.cfg.speed>1.0){
            ssml=`
              <speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
                <voice name="${speechConfig.speechSynthesisVoiceName}">
                  <prosody rate="+${this.cfg.speed*100-100}%">
                    ${input}
                  </prosody>
                </voice>
              </speak>
            `
          }else if(this.cfg.speed<1.0){
            ssml=`
              <speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
                <voice name="${speechConfig.speechSynthesisVoiceName}">
                  <prosody rate="-${100-this.cfg.speed*100}%">
                    ${input}
                  </prosody>
                </voice>
              </speak>
            `
          }else{
            ssml=`
              <speak version="1.0" xmlns="https://www.w3.org/2001/10/synthesis" xml:lang="en-US">
                <voice name="${speechConfig.speechSynthesisVoiceName}">
                  ${input}
                </voice>
              </speak>
            `
          }
          
          console.log(ssml)
          synthesizer.speakSsmlAsync(
            ssml,
            result => {
                if (result.errorDetails) {
                    console.error(result.errorDetails);
                } else {
                    console.log(JSON.stringify(result));
                }
                this.out.audio=result.audioData
                this.out_prev.audio = result.audioData

                synthesizer.close();
            },
            error => {
                console.log(error);
                synthesizer.close();
            });

        }else{
          synthesizer.speakTextAsync(
            input,
            result => {
              if (result.errorDetails) {
                console.error(result.errorDetails);
                alert("音声合成に失敗しました。正しいAPIキーを入力してください")
              }
              else{
                this.out.audio=result.audioData
                this.out_prev.audio = result.audioData
                console.log("result:",result)
                console.log("audiodata:", this.out.audio)
                synthesizer.close();
              }
            },
            error => {
                console.log(error);
                synthesizer.close();
            });
        }
      }
  },
  computed:{
    pinyinStr(){
      const arr = pinyin(this.out.text, {
        segment: "segmentit",
        group: true,
      })
      return arr.join(' ')
    }
  }
}
    

</script>

<template>

  <main>
    <div class="mx-auto px-4 py-4 max-w-screen-2xl">
      <h1 class="text-2xl lg:text-5xl font-bold my-4">
        中国語→ピンイン+音声変換
      </h1>
      <section class="overflow-hidden my-1 lg:my-0">
        <div class="border-gray-300 border rounded-lg lg:border-b-0 bg-gray-200 px-6 py-3">
          <div class="text-lg font-bold">音声合成設定</div>
          <span>APIKey: </span>
          <input v-model="apiKey" size="10" name="input" type="password" class="bg-gray-0 text-gray-800 border focus:ring ring-indigo-300 rounded outline-none transition duration-100 p-0">
        </div>
      </section>
      <section class="flex flex-col lg:border-gray-300 lg:border rounded-lg lg:flex-row justify-between lg:gap-0 gap-4 overflow-hidden">
        <div class="lg:w-5/12 p-6 lg:py-12 flex flex-col border-gray-300 border lg:border-t-0 lg:border-b-0 lg:border-l-0 rounded-lg bg-gray-100 overflow-hidden lg:rounded-none relative">
          <div class="mb-4">
            <label for="input" class="text-xl">入力(20文字まで)：</label>
            <input v-model="inputText" name="input" class="w-full bg-gray-0 text-gray-800 border focus:ring ring-indigo-300 rounded outline-none transition duration-100 px-3 py-2" />
          </div>
          <div class="mt-auto">
            <button class="inline-block custom-btn" @click="convert()">変換</button>
          </div>
        </div>

        <div class="lg:w-7/12 p-6 lg:py-12 flex flex-col border-gray-300 border rounded-lg lg:border-none lg:border-l-0 bg-gray-100 overflow-hidden lg:rounded-none relative">
          <div class="mb-4">
            <div class="text-xl sc-area">{{ pinyinStr || '　' }}</div>
            <div class="text-3xl font-bold sc-area">{{ out.text || '　'}}</div>
          </div>
          <div class="mt-auto">
            <button class="inline-block custom-btn" @click="tts()">音声</button>
          </div>
        </div>
      </section>
      
    </div>
    
    
  </main>
</template>

<style>
.custom-btn{
  @apply bg-indigo-500 hover:bg-indigo-600 active:bg-indigo-700 focus-visible:ring ring-indigo-300 text-white text-sm md:text-base font-semibold text-center rounded-lg outline-none transition duration-100 px-5 py-2;
}

.sc-area{
  font-family: -apple-system,BlinkMacSystemFont,Helvetica Neue,Helvetica,Arial,PingFang SC,Hiragino Sans GB,Microsoft YaHei,sans-serif;
}


</style>
