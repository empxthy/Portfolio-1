<template>
    <div class="container">
        <header class="header">
            <div class="chat" id="chat" v-if="showChat">
                <div class="messages">
                    <ul class="messages_text">
                        <li v-for="message in chatMessages" :key="message">
                            {{ message }}
                        </li>
                    </ul>
                </div>
                <input type="text" name="name" id="insertText" v-show="showInput" v-bind:placeholder="holder" v-model="inputText" ref="input" />
                <ul class="hints" v-show="showHints">
                    <li v-on:click="ic" class="hints-item">IC</li>
                    <li v-on:click="ooc" class="hints-item">OOC</li>
                    <li v-on:click="chat_me" class="hints-item">/me</li>
                    <li v-on:click="chat_do" class="hints-item">/do</li>
                    <li v-on:click="chat_try" class="hints-item">/try</li>
                </ul>
            </div>
        </header>
    </div>
</template>

<script>
export default {
    name: 'ChatRAGE',

    keys: {
        KEY_ENTER: 13,
        KEY_ESC: 27,
        KEY_T: 84
    },

    data() {
        return {
            showInput: false,
            showChat: false,
            showHints: false,
            chatMessages: [],
            inputText: '',
            holder: 'IC: Введите текст:',
            active: true
        }
    },
    methods: {
        addKeyListener(event){
            if(event.which === this.$options.keys.KEY_T && !this.showInput && this.activate){
                this.enableChatInput(true);
                this.$refs.input.style.visibility = 'visible';
                event.preventDefault();
                return;
            }

            if(event.which === this.$options.keys.KEY_ESC && this.showInput) {
                this.enableChatInput(false);
                this.$refs.input.style.visibility = 'hidden';
                mp.game.gameplay.setGamePaused(true);
                return;
            }

            if(event.which === this.$options.keys.KEY_ENTER && this.showInput) {
                let text = this.inputText;
    
                this.enableChatInput(false);
                this.$refs.input.style.visibility = 'hidden';
    
                if(!text.length) return;
    
                if(text.charAt(0) != "/"){
                    mp.invoke("chatMessage", text);
                    return;
                }

                text = text.substr(1)
    
                if(text.length) {
                    mp.invoke("command", text);
                }
            }
        },
        push(text) {
            if(this.chatMessages.length >= 10) {
                this.chatMessages.shift();
            }

            this.chatMessages.push(text);
        },
        clear(){
            this.chatMessages.length = 0;
        },
        enableChatInput(enable) {
            if (!this.active && enable) {
                return;
            }

            if (enable !== this.showInput) {
                mp.invoke("focus", enable);
                mp.invoke("setTypingInChatState", enable);

                this.showInput = enable;
                this.inputText = '';
                this.showChat = true;
                this.showHints = enable;
                enable && this.$nextTick().then(() => this.$refs.input.focus());
            }
        },
        activate(toggle) {
            if (!toggle && this.showInput) {
                this.enableChatInput(false);
            }
            this.active = toggle;
        },
        show(toggle) {
            if(!toggle && this.showInput) {
                this.enableChatInput(false);
            }
            this.showChat = toggle;
        },
        ic() {
            this.holder = "IC: Введите текст:";
        },
        ooc() {
            this.holder = "OOC: Введите текст:";
        },
        chat_me() {
            this.holder = "/me: Введите действие:";
        },
        chat_do() {
            this.holder = "/do: Введите действие:";
        },
        chat_try() {
            this.holder = "/try: Введите действие:";
        }
    },
    created() {
        if(mp.events){
            const api = {
                "chat:push": this.push,
                "chat:clear": this.clear,
                "chat:activate": this.activate,
                "chat:show": this.show
            }

            for(const fn in api) {
                mp.events.add(fn, api[fn]);
            }
        }
        
        document.addEventListener('keydown', this.addKeyListener);
    },
    mounted() {
        this.showChat = true;
    },
    beforeUnmount() {
        document.removeEventListener('keydown', this.addKeyListener);
    },
}
</script>
