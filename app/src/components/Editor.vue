<template>
    <div class="quick" v-on:paste="addFileFromClipboard">
        <div class="row">
            <div class="col-12 col-xl-6 mb-5">
                <slot name="top-left"/>
                <h4 class="ml-2 mb-3">Message</h4>
                <div class="card border-0 text-light mb-4 bg-darker">
                    <div class="card-body">
                        <div class="form-row mb-2">
                            <div class="col-12 col-lg-6 mb-4">
                                <label>Username</label>
                                <span class="text-muted ml-2 char-counter">{{ webhookUsername ? webhookUsername.length : 0 }} / 80</span>
                                <input v-model.trim="webhookUsername" type="text" class="form-control"
                                       placeholder="Captain Hook" maxlength="80">
                            </div>
                            <div class="col-12 col-lg-6 mb-4">
                                <label>Avatar URL
                                    <span class="ml-1 hover-tooltip" title="yeet">
                                        <i class="fas fa-question hover-tooltip-trigger"/>
                                        <span class="hover-tooltip-content bg-dark py-2 px-3 rounded">
                                            Image URLs must resolve to a valid image and respond with the correct Content-Type header.
                                            URLs ending with .png, .webp, .jpg or .jpeg are in most cases safe to use.
                                        </span>
                                    </span>
                                </label>
                                <input v-model.trim.lazy="webhookAvatarUrl" type="url" class="form-control"
                                       placeholder="https://example.com/banner.png">
                            </div>
                            <div class="col-12 mb-4">
                                <label>Content</label>
                                <span class="text-muted ml-2 char-counter">{{ content ? content.length : 0 }} / 2000</span>
                                <div class="form-check float-right">
                                    <input v-model.trim="tts" type="checkbox" class="form-check-input">
                                    <label class="form-check-label">TTS</label>
                                </div>
                                <textarea v-model="content" rows="5" class="form-control" placeholder="I like webhooks"
                                          maxlength="2000"/>
                            </div>
                            <div class="col-12">
                                <label>Files</label>
                                <span class="text-muted ml-2 char-counter">Max. 8 MB</span>
                                <div class="row mb-2 m-0">
                                <span v-for="(file, i) in files" v-bind:key="i"
                                      class="px-2 py-1 m-1 bg-dark col-auto rounded">
                                    <label class="mr-2 d-inline">{{file.name}}</label>
                                    <span class="c-pointer d-line" v-on:click="deleteFile(i)"><i
                                            class="fas fa-minus"/></span>
                                </span>
                                </div>
                                <input type="file" class="btn btn-sm btn-outline-light mr-2"
                                       v-on:change="addFileFromInput" style="max-width: 100%">
                                <div class="btn btn-sm btn-outline-light" v-on:click="clearFiles">
                                    <i class="fas fa-trash"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <h4 class="ml-2 mb-3">Embeds <span
                        class="text-muted ml-2 char-counter h6">{{ embeds.length }} / 10</span></h4>
                <div class="mb-5">
                    <div v-for="(embed, e) in embeds" v-bind:key="`embed${e}`" class="card text-light mb-3 bg-darker"
                         v-bind:style="{borderLeft: '5px solid' + embed.color}">
                        <div class="card-body">
                            <div class="float-right" style="font-size: 1.1rem">
                                <span class="mr-3 c-pointer" v-if="e !== 0" v-on:click="moveEmbedUp(e)"><i
                                        class="fas fa-chevron-up"/></span>
                                <span class="mr-3 c-pointer" v-if="e !== embeds.length - 1"
                                      v-on:click="moveEmbedDown(e)"><i class="fas fa-chevron-down"/></span>
                                <span class="mr-3 c-pointer" v-if="embeds.length < 10" v-on:click="cloneEmbed(e)"><i
                                        class="far fa-copy"/></span>
                                <span class="mr-3 c-pointer" v-on:click="deleteEmbed(e)"><i
                                        class="fas fa-minus"/></span>
                            </div>
                            <h5 data-toggle="collapse" v-bind:data-target="'#embed' + e" role="button"
                                class="overflow-auto text-nowrap">
                                <span class="mr-1">
                                    <i class="fas fa-chevron-down"/>
                                </span>
                                Embed {{ e + 1 }}
                                <span v-if="embed.title" class="text-muted">- {{ embed.title.substring(0, 15) }}</span>
                            </h5>
                            <div class="collapse mt-3" v-bind:id="'embed' + e">
                                <div class="form-row">
                                    <div class="col-12 mb-4">
                                        <label>Author</label>
                                        <span class="text-muted ml-2 char-counter">{{ embed.authorName ? embed.authorName.length : 0 }} / 256</span>
                                        <input v-model.trim="embed.authorName" type="text" class="form-control"
                                               placeholder="Doctor Who" maxlength="256">
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Author URL</label>
                                        <input v-model.trim="embed.authorUrl" type="url" class="form-control"
                                               placeholder="https://discord.club">
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Author Icon URL
                                            <span class="ml-1 hover-tooltip" title="yeet">
                                        <i class="fas fa-question hover-tooltip-trigger"></i>
                                        <span class="hover-tooltip-content bg-dark py-2 px-3 rounded">
                                            Image URLs must resolve to a valid image and respond with the correct Content-Type header.
                                            URLs ending with .png, .webp, .jpg or .jpeg are in most cases safe to use.
                                        </span>
                                    </span>
                                        </label>
                                        <input v-model.trim.lazy="embed.authorIconUrl" type="url" class="form-control"
                                               placeholder="https://example.com/banner.png">
                                    </div>
                                </div>
                                <div class="form-row">
                                    <div class="col-12 mb-4">
                                        <label>Title</label>
                                        <span class="text-muted ml-2 char-counter">{{ embed.title ? embed.title.length : 0 }} / 256</span>
                                        <input v-model.trim="embed.title" type="text" class="form-control"
                                               placeholder="My Embed" maxlength="256">
                                    </div>
                                    <div class="col-12 mb-4">
                                        <label>Description</label>
                                        <span class="text-muted ml-2 char-counter">{{ embed.description ? embed.description.length : 0 }} / 2048</span>
                                        <textarea v-model="embed.description" class="form-control"
                                                  placeholder="My first template" rows="5" maxlength="2048"/>
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>URL</label>
                                        <input v-model.trim="embed.url" type="url" class="form-control"
                                               placeholder="https://discord.club">
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Color</label>
                                        <!-- TODO: Replace with third-party color picker -->
                                        <div class="input-group">
                                            <input v-model="embed.color" type="color" class="form-control">
                                            <input type="text" v-model.trim="embed.color" class="form-control">
                                        </div>
                                    </div>
                                </div>
                                <div class="form-row">
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Image URL
                                            <span class="ml-1 hover-tooltip" title="yeet">
                                            <i class="fas fa-question hover-tooltip-trigger"/>
                                            <span class="hover-tooltip-content bg-dark py-2 px-3 rounded">
                                                Image URLs must resolve to a valid image and respond with the correct Content-Type header.
                                                URLs ending with .png, .webp, .jpg or .jpeg are in most cases safe to use.
                                            </span>
                                        </span>
                                        </label>
                                        <input v-model.trim.lazy="embed.imageUrl" type="url" class="form-control"
                                               placeholder="https://example.com/banner.png">
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Thumbnail URL
                                            <span class="ml-1 hover-tooltip" title="yeet">
                                            <i class="fas fa-question hover-tooltip-trigger"/>
                                            <span class="hover-tooltip-content bg-dark py-2 px-3 rounded">
                                                Image URLs must resolve to a valid image and respond with the correct Content-Type header.
                                                URLs ending with .png, .webp, .jpg or .jpeg are in most cases safe to use.
                                            </span>
                                        </span>
                                        </label>
                                        <input v-model.trim.lazy="embed.thumbnailUrl" type="url" class="form-control"
                                               placeholder="https://example.com/banner.png">
                                    </div>
                                </div>
                                <div class="form-row">
                                    <div class="col-12 mb-4">
                                        <label>Footer Text</label>
                                        <span class="text-muted ml-2 char-counter">{{ embed.footerText ? embed.footerText.length : 0 }} / 256</span>
                                        <input v-model.trim="embed.footerText" type="text" class="form-control"
                                               placeholder="this is the end" maxlength="256">
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Timestamp</label>
                                        <div class="form-row">
                                            <div class="col">
                                                <datetime v-model.trim="embed.timestamp" type="datetime"
                                                          input-class="form-control"/>
                                            </div>
                                            <div class="col-auto">
                                                <button class="btn btn-outline-light" type="button"
                                                        v-on:click="embed.timestamp = null">
                                                    <i class="fas fa-eraser"/>
                                                </button>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="col-12 col-lg-6 mb-4">
                                        <label>Footer Icon URL
                                            <span class="ml-1 hover-tooltip" title="yeet">
                                            <i class="fas fa-question hover-tooltip-trigger"/>
                                            <span class="hover-tooltip-content bg-dark py-2 px-3 rounded">
                                                Image URLs must resolve to a valid image and respond with the correct Content-Type header.
                                                URLs ending with .png, .webp, .jpg or .jpeg are in most cases safe to use.
                                            </span>
                                        </span>
                                        </label>
                                        <input v-model.trim.lazy="embed.footerIconUrl" type="url" class="form-control"
                                               placeholder="https://example.com/banner.png">
                                    </div>
                                </div>
                                <h5>Fields <span class="text-muted ml-2 char-counter">{{ embed.fields ? embed.fields.length : 0 }} / 25</span>
                                </h5>
                                <div v-for="(field, f) in embed.fields" v-bind:key="`field${f}`">
                                    <label>Field {{ f + 1 }}</label>
                                    <span class="text-muted ml-2 char-counter">{{ field.name ? field.name.length : 0 }} / 256</span>
                                    <span class="text-muted ml-2 char-counter">{{ field.value ? field.value.length : 0 }} / 1024</span>
                                    <div class="float-right">
                                        <span class="mr-3 c-pointer" v-if="f !== 0" v-on:click="moveFieldUp(e, f)"><i
                                                class="fas fa-chevron-up"/></span>
                                        <span class="mr-3 c-pointer" v-if="f !== embed.fields.length - 1"
                                              v-on:click="moveFieldDown(e, f)"><i class="fas fa-chevron-down"/></span>
                                        <span class="mr-3 c-pointer" v-if="embed.fields.length < 25"
                                              v-on:click="cloneField(e, f)"><i
                                                class="far fa-copy"/></span>
                                        <span class="c-pointer" v-on:click="deleteField(e, f)"><i
                                                class="fas fa-minus"/></span>
                                    </div>
                                    <div class="form-row mb-3">
                                        <div class="col-8 col-sm-10 mb-3">
                                            <input v-model.trim="field.name" type="text" class="form-control"
                                                   placeholder="Name" maxlength="256" required>
                                            <span v-if="(field.name ? field.name.length : 0) === 0" class="input-error">
                                                The field name is required
                                            </span>
                                        </div>
                                        <div class="col-4 col-sm-2 mb-3">
                                            <div class="form-check pt-2">
                                                <input v-model.trim="field.inline" type="checkbox"
                                                       class="form-check-input">
                                                <label class="form-check-label">Inline</label>
                                            </div>
                                        </div>
                                        <div class="col-12">
                                            <textarea v-model.trim="field.value" rows="2" class="form-control"
                                                      placeholder="Value" maxlength="1024" required/>
                                            <span v-if="(field.value ? field.value.length : 0) === 0"
                                                  class="input-error">
                                                The field value is required
                                            </span>
                                        </div>
                                    </div>
                                </div>
                                <div class="btn btn-sm btn-outline-light mr-2" v-on:click="clearFields(e)">
                                    <i class="fas fa-trash"/>
                                </div>
                                <div class="btn btn-sm btn-outline-light" v-on:click="addField(e)"
                                     v-bind:class="{disabled: embed.fields ? embed.fields.length >= 25 : false}">
                                    <i class="fas fa-plus"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="btn btn-sm btn-outline-secondary mr-2" v-on:click="clearEmbeds">
                        <i class="fas fa-trash"/>
                    </div>
                    <div class="btn btn-sm btn-outline-primary" v-on:click="addEmbed"
                         v-bind:class="{disabled: embeds.length >= 10}">
                        <i class="fas fa-plus"/>
                    </div>
                </div>
                <h4 class="ml-2 mb-3">Buttons <span
                        class="text-muted ml-2 char-counter h6">{{ buttons.length }} / 5</span></h4>
                <div class="mb-5">
                    <div v-for="(button, b) in buttons" :key="b" class="card text-light mb-2 bg-darker">
                        <div class="card-body">
                            <div class="float-right">
                                <span class="c-pointer" v-on:click="deleteButton(b)"><i
                                        class="fas fa-minus"/></span>
                            </div>
                            <h5 class="overflow-auto text-nowrap mb-3">
                                Button {{ b + 1 }}
                            </h5>
                            <div class="form-row mb-1">
                                <div class="col-12 mb-3">
                                    <label>Label</label>
                                    <span class="text-muted ml-2 char-counter">{{ button.label ? button.label.length : 0 }} / 80</span>
                                    <input v-model.trim="button.label" type="text" class="form-control"
                                           placeholder="Doctor Who" maxlength="80">
                                    <span v-if="(button.label ? button.label.length : 0) === 0" class="input-error">
                                                The button label is required
                                            </span>
                                </div>
                                <div class="col-12">
                                    <label>Url</label>
                                    <input v-model.trim="button.url" type="url" class="form-control"
                                           placeholder="https://discord.club">
                                    <span v-if="(button.url ? button.url.length : 0) === 0" class="input-error">
                                                The button url is required
                                            </span>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="mt-3">
                        <div class="btn btn-sm btn-outline-secondary mr-2" v-on:click="clearButtons">
                            <i class="fas fa-trash"/>
                        </div>
                        <div class="btn btn-sm btn-outline-primary" v-on:click="addButton"
                             v-bind:class="{disabled: buttons.length >= 5}">
                            <i class="fas fa-plus"/>
                        </div>
                    </div>
                </div>
            </div>
            <div class="col-12 col-xl-6">
                <h4 class="ml-2 mb-3">Preview</h4>
                <div class="card preview mb-5 bg-discordbg">
                    <div class="card-body px-1 py-3">
                        <preview v-bind:data="fullData"/>
                    </div>
                </div>
                <h4 class="ml-2 mb-3">JSON Code</h4>
                <div class="card bg-darker mb-5">
                    <div class="card-body px-3 py-3">
                        <textarea v-model.lazy="jsonCode" class="form-control mb-2" rows="10"/>
                        <span class="text-danger">{{ jsonError }}</span>
                        <div class="float-right">
                            <button class="btn btn-outline-light mr-2" v-on:click="shareJSON"
                                    :class="{disabled: jsonIsEmpty}" :disabled="jsonIsEmpty" type="button">
                                Share
                            </button>
                            <button class="btn btn-outline-light mr-2" v-on:click="exportJSON" type="button">
                                Download
                            </button>
                            <slot name="save"/>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div class="modal fade" id="shareModal" tabindex="-1" role="dialog">
            <div class="modal-dialog" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">Shared Message</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <div class="modal-body">
                        <p>
                            This link will be valid for 24 hours and only gives other user access to clone your message.
                            There is no way for them to edit your copy of the message.
                        </p>
                        <a :href="lastShareLink" target="_blank">{{lastShareLink}}</a>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-primary" v-on:click="copyShareLink">Copy Link</button>
                        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
                    </div>
                </div>
            </div>
        </div>
        <confirmation
                text="Do you want to load the JSON-Code from the shared message? This will overwrite your existing message!"
                ref="shareOverwriteConfirmation"/>
    </div>
</template>

<script>
    import Preview from './Preview.vue'
    import {Datetime} from 'vue-datetime'
    import 'vue-datetime/dist/vue-datetime.css'
    import $ from 'jquery'
    import Confirmation from "@/components/Confirmation";

    export default {
        name: 'Editor',
        components: {Preview, Datetime, Confirmation},
        props: ['initData'],
        data() {
            return {
                webhookUsername: "",
                webhookAvatarUrl: "",
                content: "",
                tts: undefined,
                files: [],
                embeds: [],
                buttons: [],

                jsonError: "",

                lastShareId: ""
            }
        },
        created() {
            if (this.initData) {
                this.fullData = this.initData
            }
            this.$emit('dataUpdate', this.fullData)
        },
        mounted() {
            const shareid = this.$route.query.share
            if (shareid) {
                this.$refs.shareOverwriteConfirmation.open().then(confirmed => {
                    if (confirmed) {
                        this.api.getShare(shareid)
                            .then(resp => resp.json())
                            .then(data => this.setJSON(data.json))
                    }
                })
            }
        },
        methods: {
            getJSON() {
                function formatFix(text) {
                    if (!text) {
                        return text
                    }
                    return text.replace(/\u200b>/gm, '>')
                }

                let embeds = []
                for (let embed of this.embeds) {
                    let fields = []
                    if (embed.fields) {
                        for (let field of embed.fields) {
                            fields.push({
                                name: field.name,
                                value: formatFix(field.value),
                                inline: field.inline
                            })
                        }
                    }

                    embeds.push({
                        title: embed.title,
                        color: embed.color ? parseInt(embed.color.substring(1), 16) : undefined,
                        description: formatFix(embed.description),
                        timestamp: embed.timestamp,
                        url: embed.url,
                        author: {
                            name: embed.authorName,
                            url: embed.authorUrl,
                            icon_url: embed.authorIconUrl
                        },
                        image: {
                            url: embed.imageUrl
                        },
                        thumbnail: {
                            url: embed.thumbnailUrl
                        },
                        footer: {
                            text: embed.footerText,
                            icon_url: embed.footerIconUrl
                        },
                        fields: fields
                    });
                }

                let components = []
                let buttons = []
                for (let button of this.buttons) {
                    buttons.push({
                        type: 2,
                        style: 5,
                        label: button.label,
                        url: button.url
                    })
                }

                if (buttons.length > 0) {
                    components.push({type: 1, components: buttons})
                }

                const data = {
                    username: this.webhookUsername,
                    avatar_url: this.webhookAvatarUrl,
                    content: formatFix(this.content),
                    tts: this.tts,
                    embeds: embeds,
                    components: components
                }
                return data
            },
            setJSON(data) {
                let embeds = []
                let buttons = []

                // Legacy support
                if (data.embed) {
                    if (data.embeds) {
                        data.embeds.push(data.embed)
                    } else {
                        data.embeds = [data.embed]
                    }
                }

                if (data.embeds) {
                    for (let embed of data.embeds) {
                        let parsed = {
                            title: embed.title,
                            color: embed.color ? "#" + embed.color.toString(16) : "#000",
                            description: embed.description,
                            timestamp: embed.timestamp,
                            tts: embed.tts,
                            url: embed.url,
                            fields: embed.fields
                        }

                        if (embed.author) {
                            parsed.authorName = embed.author.name
                            parsed.authorUrl = embed.author.url
                            parsed.authorIconUrl = embed.author.icon_url
                        }

                        if (embed.image) {
                            parsed.imageUrl = embed.image.url
                        }

                        if (embed.thumbnail) {
                            parsed.thumbnailUrl = embed.thumbnail.url
                        }

                        if (embed.footer) {
                            parsed.footerText = embed.footer.text
                            parsed.footerIconUrl = embed.footer.icon_url
                        }

                        embeds.push(parsed);
                    }
                }

                if (data.components && data.components.length > 0) {
                    let row = data.components[0]
                    if (row && row.components) {
                        for (let button of row.components) {
                            buttons.push({
                                label: button.label,
                                url: button.url
                            })
                        }
                    }
                }

                this.webhookUsername = data.username;
                this.webhookAvatarUrl = data.avatar_url;
                this.content = data.content;
                this.embeds = embeds;
                this.buttons = buttons
            },
            addEmbed() {
                if (this.embeds.length >= 10) {
                    return
                }
                this.embeds.push({color: "#000"})
                setTimeout(() => {
                    $(`#embed${this.embeds.length - 1}`).addClass('show')
                }, 100)
            },
            clearEmbeds() {
                this.embeds = []
            },
            deleteEmbed(i) {
                this.embeds.splice(i, 1)
            },
            moveEmbedUp(i) {
                this.embeds.splice(i - 1, 0, this.embeds.splice(i, 1)[0])
            },
            moveEmbedDown(i) {
                this.embeds.splice(i + 1, 0, this.embeds.splice(i, 1)[0])
            },
            cloneEmbed(i) {
                if (this.embeds.length >= 10) {
                    return
                }
                this.embeds.splice(i + 1, 0, JSON.parse(JSON.stringify(this.embeds[i])))
            },
            addField(e) {
                let embed = this.embeds[e];
                if (!embed.fields) {
                    embed.fields = []
                }
                if (embed.fields.length >= 25) {
                    return
                }
                embed.fields.push({})
                embed.content = embed.content ? embed.content : undefined
                this.$forceUpdate() // idk why it doesn't work without this (maybe too much nesting)
            },
            clearFields(e) {
                this.embeds[e].fields = []
                this.$forceUpdate() // idk why it doesn't work without this (maybe too much nesting)
            },
            deleteField(e, i) {
                let embed = this.embeds[e];
                if (embed.fields) {
                    embed.fields.splice(i, 1)
                }
                this.$forceUpdate() // idk why it doesn't work without this (maybe too much nesting)
            },
            moveFieldUp(e, i) {
                let embed = this.embeds[e];
                embed.fields.splice(i - 1, 0, embed.fields.splice(i, 1)[0])
                this.$forceUpdate()
            },
            moveFieldDown(e, i) {
                let embed = this.embeds[e];
                embed.fields.splice(i + 1, 0, embed.fields.splice(i, 1)[0])
                this.$forceUpdate()
            },
            cloneField(e, i) {
                let embed = this.embeds[e];
                if (embed.fields.length >= 25) {
                    return
                }
                embed.fields.splice(i + 1, 0, JSON.parse(JSON.stringify(embed.fields[i])))
                this.$forceUpdate()
            },
            addFile(file) {
                let reader = new FileReader()
                reader.readAsArrayBuffer(file)
                reader.onload = re => {
                    let nameParts = file.name.split(".")
                    let name = nameParts.slice(0, -1);
                    let extension = nameParts[nameParts.length - 1];
                    let existingNames = this.files.map(f => f.name)
                    while (existingNames.includes(`${name}.${extension}`)) {
                        name += "_"
                    }

                    this.files.push({
                        name: `${name}.${extension}`,
                        content: new Blob([re.target.result]),
                        size: re.target.result.byteLength
                    })
                }
            },
            addFileFromInput(e) {
                let file = e.target.files[0]
                e.target.value = null;
                if (file) {
                    this.addFile(file)
                }
            },
            addFileFromClipboard(e) {
                let file = e.clipboardData.files[0]
                if (file) {
                    this.addFile(file)
                }
            },
            deleteFile(i) {
                this.files.splice(i, 1)
            },
            clearFiles() {
                this.files = []
            },
            addButton() {
                if (this.buttons.length >= 5) return
                this.buttons.push({})
            },
            clearButtons() {
                this.buttons.splice(0, this.buttons.length)
            },
            deleteButton(b) {
                this.buttons.splice(b, 1)
            },
            exportJSON() {
                let element = document.createElement('a');
                element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(this.jsonCode));
                element.setAttribute('download', 'message.json');
                element.style.display = 'none';
                document.body.appendChild(element);
                element.click();
                document.body.removeChild(element);
            },
            shareJSON() {
                this.api.createShare(this.getJSON())
                    .then(resp => resp.json())
                    .then(data => {
                        this.lastShareId = data.id
                        $('#shareModal').modal()
                    })
            },
            copyShareLink() {
                navigator.clipboard.writeText(this.lastShareLink)
            }
        },
        computed: {
            jsonCode: {
                get() {
                    return JSON.stringify(this.getJSON(), null, 2)
                },
                set(newValue) {
                    let data;
                    try {
                        data = JSON.parse(newValue);
                    } catch (SyntaxError) {
                        this.jsonError = "Invalid JSON-Code, unable to apply it.";
                        return;
                    }
                    this.jsonError = "";
                    this.setJSON(data);
                }
            },
            fullData: {
                get() {
                    return {
                        json: this.getJSON(),
                        files: this.files
                    }
                },
                set(data) {
                    this.setJSON(data.json ? data.json : {})
                    this.files = data.files ? data.files : []
                }
            },
            api() {
                return this.$store.state.api
            },
            lastShareLink() {
                return `${process.env.VUE_APP_SITE_HOST}/share/${this.lastShareId}`
            },
            jsonIsEmpty() {
                return !this.webhookUsername && !this.webhookAvatarUrl && !this.content && this.embeds.length === 0
            }
        },
        watch: {
            $data: {
                handler() {
                    this.$emit('dataUpdate', this.fullData)
                },
                deep: true
            },
            initData(newValue) {
                if (this.initData) {
                    this.fullData = newValue
                }
            }
        }
    }
</script>

<style scoped lang="scss">
    .preview {
        box-shadow: 0 0 2px #212121;
    }

    .char-counter {
        font-style: italic;
        font-size: 0.8em;
    }

    .input-error {
        font-size: 0.9em;
        color: red;
        opacity: 0.9;
        font-style: italic;
    }
</style>
<style>
    .vdatetime-popup {
        background-color: black;
        color: white;
    }

    .vdatetime-calendar__month__day:hover > span > span {
        background: #eee;
        color: black;
    }
</style>
