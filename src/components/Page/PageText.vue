<template>
    <p class="prattle" v-html="filteredPrattle" v-if="textType == 'prattle' && !!content"></p>

    <div class="log" :class="{logHidden: logHidden}" v-else-if="textType == 'log'">
		<button class="logButton" @click="loggle()">
            {{ logButtonText }}
		</button>
		<p class="logContent" v-html="content.replace(/\|.*?\| *\<br \/\>/, '')"></p>
	</div>

    <div class="authorlog" v-else-if="textType == 'authorlog'">
		<p class="logContent" v-html="content.replace(/\|.*?\| *\<br ?\/?\>/, '')"></p>
	</div>

</template>

<script>
export default {
    name: 'pageText',
    props: ['pageId', 'content'],
    data() {
        return {
            usePurpleLinks: false,
            logHidden: true,
            fullwidthImages: [
                'andthenightbeforechristmas.jpg',
                'areyounext.gif',
                'corpsesmooch.gif',
                'THIS.gif',
                'IS.gif',
                'STUPID.gif',
                'WHOSTHISDOUCHEBAG.gif',
                'nextbowl.gif',
                'nextbowl2.gif',
                'nextbowl3.gif',
                'nextbowl4.gif',
                'HELPHIM.gif',
                'weirdhonk.gif',
                'OHBOY.gif',
                'wizordofoz.jpg'
            ]
        }
    },
    computed: {
        textType() {
            return this.getTextType(this.content)
        },
        logButtonText() {
            let text = this.content.match(/\|(.*?)\|/)[1]
            let state = (this.logHidden) ? 'Show ' : 'Hide '

            if (/P4SSWORD/i.test(text)){
                return text
            }
            else if (/trkstrlog/i.test(text)){
                text = 'tricksterlog'
            }
            else if (/sriousbiz/i.test(text)){
                text = 'serious business'
            }
            return state + text.toLowerCase()
        },
        filteredPrattle() {
            let result = this.content
            if (/<img src="/.test(result)){
                this.fullwidthImages.some(img => {
                    result = result.replace(`${img}" `, `${img}" class='fullWidth'`)
                })
            }
            
            return result
        },
        purpleLinkWatcher() {
            return this.$localData.temp.visited.toString()
        }
    },
    methods: {
        loggle: function() {
            this.logHidden = !this.logHidden
        },
        getTextType(content){
            if (!content) return null

            if (/^\|AUTHORLOG\|/.test(content)){
                return "authorlog"
            }
            else if (/^\|.*?\|/.test(content)){
                return "log"
            }
            else{
                return "prattle"
            }
        },
        filterDOM() {
            if (this.$el.nodeType !== 8){
                let links = this.$el.getElementsByTagName('A')
                for(let i = 0;i < links.length; i++) {
                    let filteredLink = this.$filterURL(links[i].href)
                    links[i].href = filteredLink
                    if (this.usePurpleLinks && this.$localData.temp.visited.includes(filteredLink.slice(-6))) links[i].classList.add('visited')
                }
                let images = this.$el.getElementsByTagName('IMG')
                for(let i = 0;i < images.length; i++) {
                    images[i].src = this.$mspaURL(images[i].src)
                    images[i].ondragstart = (e) => {
                        e.preventDefault()
                        e.dataTransfer.effectAllowed = 'copy'
                        require('electron').ipcRenderer.send('ondragstart', this.$mspaFileStream(images[i].src))
                    }
                }
            }
        }
    },
    mounted() {
        if (this.$localData.settings.openLogs) {
            //Manual exception for pre-ministrife fakeout
            this.logHidden = this.pageId == '007326' 
        }
        
        // Purple visited links were originally intended to function on every page, but it was too much of a pain in the ass
        // So instead, you get this awful fucking hyperspecific implementation that handles precisely one page that requires them and nothing else
        // ur welcom
        if (this.pageId == '009386') this.usePurpleLinks = true
        if (['009387','009393','009397','009400','009402','009408','009410','009413'].includes(this.pageId) && !this.$localData.temp.visited.includes(this.pageId)) this.$localData.temp.visited.push(this.pageId) 

        this.filterDOM()
    },
    watch: {
        'purpleLinkWatcher'() {
            if (this.usePurpleLinks) this.filterDOM()
        }
    }
}
</script>

<style scoped lang="scss">
    ::v-deep {
        image-rendering: pixelated;
        
        a {
            color: var(--page-links);
            &.visited {
                color: var(--page-links-visited);
            }
        }
    }

    .prattle {
        width: 600px;
        text-align: center;
        margin: 0 0 30px 0;
        
        /deep/ img.fullWidth {
            position: relative;
            left: -25px;
        }
    }

    .authorlog {
        width: 600px;
        margin: 0 0 30px 0;
        border: 3px solid var(--page-pageBorder, var(--page-pageFrame));
        background: white;
        padding: 1px;
        text-align: center;
        align-self: center;
        .logContent{
            padding: 15px 5%;
            text-align: left;
        }
    }

    .log {
        width: 600px;
        margin: 0 0 30px 0;
        border: 1px dashed var(--page-log-border);
        background: var(--page-log-bg);
        padding: 1px;
        text-align: center;
        align-self: center;
        
        &.highContrast {
            background: #ffffff;
        }
        button {
            text-transform: capitalize;
        }
        
        .logContent{
            padding: 15px 5%;
            text-align: left;
            line-height: 1.35;
        }

        &.logHidden {
            .logContent {
                display: none; 
            }
        }
    
    }
</style>


