<template>
    <main id="app">
        <!-- TopHead is the header with the information about the app -->
        <TopHead v-if="app && messages.length > 0" :app="app"></TopHead>
        <section class="container chat-container">

            <!-- Welcome component is for onboarding experience and language picker -->
            <Welcome v-if="app && messages.length == 0" :app="app"></Welcome>

            <!-- Messages Table -->
            <section class="messages" v-else>
                <table v-for="m in messages" class="message">
                    <tr>
                        <!-- My message -->
                        <td><Bubble :text="m.queryResult.queryText"  from="me" /></td>
                    </tr>
                    
                    <!-- Component iterator (Dialogflow Gateway Feature) -->
                    <tr v-for="component in m.queryResult.fulfillmentMessages">

                        <td>
                            <!-- Default / Webhook bubble -->
                            <!--{{component}}-->
                            <Bubble :text="component.content" :mp3url="component.mp3url"  v-if="component.name == 'MP3'" />

                            <Bubble :text="component.content" :mp3url="component.mp3url"  v-if="component.name == 'DEFAULT'" />

                            <Bubble :text="component.content" :imageUrl="component.imageUrl"  v-if="component.name == 'image'" />

                            <!--&lt;!&ndash; Simple Response &ndash;&gt;-->
                            <Bubble :text="component.content.displayText || component.content.textToSpeech" v-if="component.name == 'SIMPLE_RESPONSE'" />

                            <!-- Card -->
                            <Card :title="component.content.title" :subtitle="component.content.subtitle" :image="component.content.image" :text="component.content.formattedText" :button="component.content.buttons[0]" v-if="component.name == 'CARD'" />
                            
                            <!-- Carousel layout and cards -->
                            <div class="carousel" v-if="component.name == 'CAROUSEL_CARD'">
                                <Card v-for="card in component.content" @click.native="send(card.info.key)" :key="card.info.key" :title="card.title" :image="card.image" :subtitle="card.subtitle" :text="card.description" />
                            </div>

                            <!-- List -->
                            <List @select="send" :items="component.content.items" :title="component.content.title" v-if="component.name == 'LIST'" />

                            <!-- Webhook Image -->
                            <Picture v-if="component.name == 'IMAGE'" :image="component.content" />
                        </td>
                    </tr>
                </table>
                <table class="message" v-if="loading">
                    <tr>
                        <!-- My message (Loading) -->
                        <td><Bubble from="me" loading="true" /></td>
                    </tr>
                    <tr>
                        <!-- Default / Webhook bubble (Loading) -->
                        <td><Bubble loading="true" /></td>
                    </tr>
                </table>
            </section>
        </section>

        <!-- #bottom is the anchor, we need, when new messages arrive, to scroll down -->
        <div id="bottom"></div>

        <!-- ChatInput is made for submitting queres and displaying suggestions -->
        <ChatInput @submit="send" :suggestions="suggestions"></ChatInput>

        <!-- Audio toggle (on the top right corner), used to toggle the audio output, default mode is defined in the settings -->
        <div :aria-label="(config.i18n[lang()] && config.i18n[lang()].inputTitle) || config.i18n[config.app.fallback_lang].muteTitle" :title="(config.i18n[lang()] && config.i18n[lang()].inputTitle) || config.i18n[config.app.fallback_lang].muteTitle" class="audio-toggle" @click="muted = !muted">
            <i aria-hidden="true" class="material-icons" v-if="!muted">volume_up</i>
            <i aria-hidden="true" class="material-icons" v-else>volume_off</i>
        </div>
    </main>
</template>

<style lang="sass">
@import url('https://fonts.googleapis.com/css?family=Roboto:400,500,700')

body
    margin: 0
    padding: 0
    font-family: Roboto, sans-serif
    font-display: swap
    background-color: white

.container
    max-width: 500px
    margin-left: auto
    margin-right: auto
    padding: 16px
    position: relative

@font-face
    font-family: 'Material Icons'
    font-style: normal
    font-weight: 400
    font-display: swap
    src: url(https://fonts.gstatic.com/s/materialicons/v42/flUhRq6tzZclQEJ-Vdg-IuiaDsNcIhQ8tQ.woff2) format('woff2')

.material-icons
    font-family: 'Material Icons'
    font-weight: normal
    font-style: normal
    font-size: 24px
    line-height: 1
    letter-spacing: normal
    text-transform: none
    display: inline-block
    white-space: nowrap
    word-wrap: normal
    direction: ltr
    -webkit-font-feature-settings: 'liga'
    -webkit-font-smoothing: antialiased
</style>

<style lang="sass" scoped>
.chat-container
    padding-top: 60px
    padding-bottom: 125px

.message
    width: 100%




.audio-toggle
    position: fixed
    top: 0
    right: 0
    margin: 8px
    z-index: 999
    padding: 10px
    background-color: #F1F3F4
    border-radius: 50%
    width: 24px
    height: 24px
    cursor: pointer
    color: #202124

.carousel
    overflow-x: scroll
    overflow-y: hidden
    white-space: nowrap
    -webkit-overflow-scrolling: touch
    padding-bottom: 20px
    padding-left: 10px
</style>

<script>




    import Welcome from './../Welcome/Welcome.vue'
import TopHead from './../Partials/TopHead.vue'
import ChatInput from './../Partials/ChatInput.vue'

import Bubble from './../RichComponents/Bubble.vue'
import Card from './../RichComponents/Card.vue'
import List from './../RichComponents/List.vue'
import Picture from './../RichComponents/Picture.vue'

import * as uuidv1 from 'uuid/v1'

export default {
    name: 'app',
    components: {
        Welcome,
        TopHead,
        ChatInput,
        Bubble,
        Card,
        List,
        Picture
    },
    data() {
        return {
            app: null,
            messages: [],
            language: '',
            session: '',
            muted: this.config.app.muted,
            loading: false,
            mp3url: '',
            imageUrl: '',
            hey: 'hey'
        };
    },
    // playSound (sound) {
    //     if(sound) {
    //         var audio = new Audio(sound);
    //         audio.play();
    //     }
    // },
    created(){
        /* If history is enabled, the messages are retrieved from localStorage */
        if(this.history() && localStorage.getItem('message_history') !== null){
            this.messages = JSON.parse(localStorage.getItem('message_history'))
        }

        /* Session should be persistent (in case of page reload, the context should stay) */
        if(this.history() && localStorage.getItem('session') !== null){
            this.session = localStorage.getItem('session')
        }

        else {
            this.session = uuidv1();
            if(this.history()) localStorage.setItem('session', this.session)
        }

        /* Cache Agent (this will save bandwith) */
        if(this.history() && localStorage.getItem('agent') !== null){
            this.app = JSON.parse(localStorage.getItem('agent'))
        }
    },
    computed: {
        /* The code below is used to extract suggestions from last message, to display it on ChatInput */
        suggestions(){
            if(this.messages.length > 0){
                let last_message = this.messages[this.messages.length - 1].queryResult.fulfillmentMessages;
                let suggestions = [];

                for (let component in last_message){
                    if (last_message[component].name === 'SUGGESTIONS') suggestions.text_suggestions = last_message[component].content
                    if (last_message[component].name === 'LINK_OUT_SUGGESTION') suggestions.link_suggestion = last_message[component].content
                }

                return suggestions
            }
            
            else {
                return {
                    text_suggestions: this.config.app.start_suggestions // <- if no messages are present, return start_suggestions, from config.js to help user figure out what he can do with your application
                }
            }
        }
    },
    watch: {
        /* This function is triggered, when new messages arrive */
        messages(messages){
            if(this.history()) localStorage.setItem('message_history', JSON.stringify(messages)) // <- Save history if the feature is enabled
        },
        /* This function is triggered, when request is started or finished */
        loading(){
            setTimeout(() => {
                let app = document.querySelector('#app'); // <- We need to scroll down #app, to prevent the whole page jumping to bottom, when using in iframe
                app.querySelector('#bottom').scrollIntoView({ 
                    behavior: 'smooth' 
                })
            }, 2) // <- wait for render (timeout) and then smoothly scroll #app down to #bottom selector, used as anchor
        },
        /* You don't need the function below. It's only for my cloud, to manage the SEO */
        app(agent){
            if(window.location.host.includes("cloud.ushakov.co")){
                document.querySelector("title").innerText = agent.displayName
                document.querySelector("meta[name=description]").content = agent.description
                document.querySelector("link[rel=canonical]").href = location.href
                document.querySelector("meta[name=application-name]").content = agent.displayName
                document.querySelector("link[rel=icon]").href = agent.avatarUri
                document.querySelector("link[rel=apple-touch-icon]").href = agent.avatarUri
                document.querySelector("meta[name=msapplication-TileImage]").content = agent.avatarUri
                document.querySelector("meta[name=apple-mobile-web-app-title]").content = agent.displayName
                document.querySelector("meta[property=og\\:title]").content = agent.displayName
                document.querySelector("meta[property=og\\:image]").content = agent.avatarUri
                document.querySelector("meta[property=og\\:description]").content = agent.description
                document.querySelector("meta[property=og\\:url]").content = location.href
                document.querySelector("meta[name=twitter\\:title]").content = agent.displayName
                document.querySelector("meta[name=twitter\\:image]").content = agent.avatarUri
                document.querySelector("meta[name=twitter\\:description]").content = agent.description
            }
        }
    },
    methods: {
        send(q){
            console.log("sending =>" + q);
            let request = {
                "text": q,
                "languageCode": this.lang()
            }; // <- this is how a Dialogflow request look like

            this.loading = true;

            // Make the request to gateway with formatting enabled */
            fetch('https://b41b1a7e.ngrok.io/kor/getBotResponse', {method: 'POST', headers: {'content-type': 'application/json'}, body: JSON.stringify(request)})
            .then(response => {
                console.log("response.json() = ");
                return response.json();
            })
            .then(response => {
                console.log(response.queryResult.fulfillmentMessages);
                this.messages.push(response);
                this.handle(response); // <- trigger the handle function (explanation below)
                this.loading = false
                //console.log(response) // <- (optional) log responses
            });
        },
        handle(response){

        }
    }
}
</script>