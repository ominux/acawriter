<template>
    <div class="container-fluid">
        <div class="row">
            <div class="col-md-6"><h3 v-if="preSetAssignment">{{preSetAssignment.name}}</h3></div>
        </div>
        <div class="row">
            <div class="col-md-6"><small>This is an example piece of writing to experiment with. You can edit this text (nothing is saved).</small></div>
            <div class="col-md-6"><small>Click Get Feedback to see the automatic feedback on the right: click the Key for more information about the icons and highlights.
                When you're ready, go to <a href="/home">My Dashboard</a> and create your own document.</small></div>
        </div>
        <div v-if="admin" class="row">
            <div class="col-md-12">
                <div class="card bg-secondary text-white">
                    <div class="card-block p-3">
                        <div class="row">
                            <div class="col-md-4"><label for="faculty">Faculty</label><input class="form-control" type="text" id="faculty" v-model="example.faculty" /></div>
                            <div class="col-md-4"><label for="faculty">Title</label><input class="form-control" type="text" id="title" v-model="example.title" /></div>
                            <div class="col-md-4"><label for="genre">Genre</label>
                                <select class="form-control" id="genre" v-model="example.genre">
                                    <optgroup v-for= "(item, key) in features" :label="key">
                                        <option v-for = "val in item" :value="val.id">{{val.name}}</option>
                                    </optgroup>

                                <!-- <option value="">Select</option>
                                <option value="2">Reflective</option>
                                <option value="1">Analytic</option> -->
                                </select>
                            </div>
                        </div>
                        <div class="row">
                            <div class="col-md-12"><label for="summary">Summary</label><input class="form-control" type="text" v-model="example.summary" id="summary" /></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <br />
        <div class="row">
            <div class="col-md-12">
                <div v-if="errors && errors.length" class="col-md-12 alert alert-danger" role="alert">
                    <ul>
                        <li v-for="error in errors">{{error.message}}</li>
                    </ul>
                </div>
            </div>
        </div>
        <div class="row editWrapper">
            <div id="sidebar" class="active" v-bind:class="this.attributes.grammar == 'analytical'? 'ana' : 'ref'">
                <div class="p-3 bg-uts-primary text-white"><i class="fa fa-info-circle" aria-hidden="true"></i> Key
                    <i class="fa fa-times-circle pull-right" aria-hidden="true" id="sidebarCollapseTwice"></i>
                    <i class="fa fa-window-restore pull-right" aria-hidden="true" id="extendOut"></i>
                </div>

                <div class="col-md-12 col-xs-12" v-for="rule in feedback.rules">
                    <h6 class="card-subtitle p-4" v-if="rule.custom">{{rule.custom}}</h6>
                    <div v-for="msg in rule.message">
                        <div class="row" v-for="(m,id) in msg">
                            <div class="col-md-1"><input type="checkbox" v-bind:id="id" v-bind:value="id" checked="checked"></div>
                            <div class="col-md-10"><span v-bind:class="id"></span>&nbsp;<span v-html="m"></span></div>
                        </div>
                    </div>
                    <hr />
                </div>
            </div>


            <!-- start content -->
            <div id="content" class="col-md-12">
                <div class="card">
                    <div class="card-header bg-primary text-white">
                        <div class="row">
                            <div class="col-md-2">Example Text Analyser</div>
                            <div class="col-md-2 text-right">
                                <span class="text-white" v-if="auto!=''"><small>{{auto}}</small></span>
                            </div>
                            <div class="col-md-8">
                                <div class="btn-group pull-right" role="group" aria-label="Button group with nested dropdown">
                                    <button type="button" class="btn btn-primary" v-on:click="fetchFeedback()"><i class="fa fa-cloud-download"  aria-hidden="true"></i> Get Feedback</button>&nbsp;
                                    <button type="button" class="btn btn-primary btn-sm" v-if="admin" v-on:click="storeAnalysedDrafts()"><i class="fa fa-floppy-o" aria-hidden="true"></i> Save</button>&nbsp;
                                    <button type="button" id="sidebarCollapse" class="btn btn-primary"><i class="fa fa-info-circle" aria-hidden="true"></i> Key</button>
                                </div>
                            </div>

                        </div>

                    </div>
                    <div class="card-body">
                        <div class="row">
                            <div class="col-md-6">
                                <div id="editor">
                                    <vue-editor v-model="editorContent" :editorToolbar="customToolbar"></vue-editor>
                                </div>
                            </div>
                            <!--- Reflective feedback --->
                            <div class="col-md-6 bg-light" v-bind:class="this.attributes.grammar == 'reflective'? 'activeClass' : 'nonactive'" v-if="this.attributes.grammar == 'reflective'">
                                <reflective-result></reflective-result>
                            </div>
                            <!-- end of reflective -->


                            <!--- Analytic feedback --->
                            <div class="col-md-6 bg-light" v-bind:class="this.attributes.grammar == 'analytical'? 'activeClass' : 'nonactive'" v-if="this.attributes.grammar == 'analytical'">
                                <analytic-result></analytic-result>
                            </div>
                            <!-- end of analytics -->

                        </div>
                    </div>
                </div>
            </div>

        </div>

    </div>

</template>

<script>
    /**
     commented out - license needed
     import VueFroala from 'vue-froala-wysiwyg';
     Vue.use(VueFroala);
     *
     */
    import { VueEditor } from 'vue2-editor';
    import moment from 'moment';
    import store from '../store';
    import { mapState, mapActions, mapGetters} from 'vuex';
    import  Reflective from './analyser/Reflective.vue';
    import  Analytic from './analyser/Analytic.vue';

    export default {
        components: {
            VueEditor,
            reflectiveResult: Reflective,
            analyticResult: Analytic
        },
        name: 'editor',
        props:['ex', 'role', 'ext'],
        store,
        data () {
            return {
                editorContent: 'Edit Your Content Here!',
                loading: 0,
                tap:[],
                errors:[],
                counter:0,
                tempIds:[],
                auto:'',
                autosave:'',
                example:{
                    faculty:'',
                    title:'',
                    summary:'',
                    genre:1
                },
                splitText:[],
                quickTags:'',
                customToolbar: [
                    ['bold', 'italic', 'underline'],
                    [{ 'list': 'ordered'}, { 'list': 'bullet' }],
                ],
                cssSpec: {
                    inline :['link2me'],
                    iconic :['context', 'challenge', 'change', 'metrics', 'affect'],
                    inText :['affect', 'epistemic','modall']
                },
                initFeedback:false
            }
        },
        mounted () {
            if(this.initFeedback) {
                this.fetchFeedback();
            }
        },
        created() {
            this.auto = '';
            //setInterval(this.storeAnalysedDrafts, 900000);
            // setInterval(this.quickCheck, 300000);
        },
        computed: {
            reflective: function() {
                return this.attributes.feedbackOpt == 'reflective' ? 'display:inline': '';
            },
            analytic: function() {
                return this.attributes.feedbackOpt == 'analytical' ? 'display:inline': '';
            },
            ...mapGetters({
                feedback: 'currentFeedback',
                processing: 'loadingStatus'
            }),
            admin: function() {
                return this.role;
            },
            preSetAssignment: function() {
                if(this.ex) {
                    return JSON.parse(this.ex);
                } else {
                    return false;
                }
            },
            features: function() {
              return JSON.parse(this.ext);
            },
            grammar: function() {
                var self =this;
                let gram ='';
                for( var [k ,v] of Object.entries(this.features)) {
                    v.forEach(function(item){
                        if(item.id == self.example.genre)  {
                            gram =  k.toLowerCase();
                        }
                    });
                }
                return gram;
            },
            attributes: function() {
                if(this.preSetAssignment) {
                    if(this.preSetAssignment.raw_response) {
                        this.editorContent = this.preSetAssignment.text_input;

                        let data = {'savedFeed':JSON.parse(this.preSetAssignment.raw_response)};
                        this.$store.dispatch('PRELOAD_FEEDBACK',data);
                        this.initFeedback = false;
                    }
                    let feature = this.preSetAssignment.feature;
                    return {
                        feedbackOpt:feature.grammar.toLowerCase() == 'analytical' ? 'a_01': 'r_01',
                        grammar: feature.grammar.toLocaleLowerCase(),
                        feature: feature.id,
                        storeDraftJobRef: Math.random().toString(36).substring(7),
                        initFeedback:this.initFeedback

                    };
                } else {
                    return {
                        feedbackOpt:this.example.genre== 1 ? 'a_01' : 'r_01',
                       // grammar: this.example.genre == 1 ? 'analytic' : 'reflective',
                        grammar: this.grammar,
                        feature:this.example.genre,
                        storeDraftJobRef: Math.random().toString(36).substring(7),
                        initFeedback:this.initFeedback
                    };
                }
                //setInterval(this.storeAnalysedDrafts('auto'), 900000);
            },
            rulesClasses: function() {
                let rules = [];
                rules = this.feedback.rules.map(function (rule, idx) {
                    return rule.css.map(function (cl) {
                        return cl;
                    });
                });
                let classes = [].concat(...rules);
                return classes;
            }


        },
        watch :{
        },
        methods: {
            fetchFeedback() {
                this.errors=[];
                //this.autoCheck = true;
                if(this.feedbackOpt!=='') {
                    // let data = {'tap': this.tap, 'txt':'', 'action': 'fetch', 'extra': this.attributes};
                    let data = {'txt':this.editorContent, 'action': 'fetch', 'extra': this.attributes};
                    this.$store.dispatch('LOAD_FEEDBACK',data);
                    //this.autoCheck = false;
                } else {
                    this.$data.errors.push({'message':'Please select feedback type'});
                }
            },storeAnalysedDrafts() {
                this.$data.auto='processing....';
                this.attributes.initFeedback = true;
                let data = {'txt':this.editorContent, 'action': 'store', 'extra': this.attributes,'feedback':this.feedback, 'other':this.example};
                axios.post('/example/store', data)
                    .then(response => {
                        this.$data.auto = 'Stored : '+ moment().format('DD/MM/YYYY hh:mma');
                    })
                    .catch(e => {
                        this.$data.errors.push(e)
                    });
            },
            getGrammar: function(idx) {
                for( var [k ,v] of Object.entries(this.features)) {
                    v.forEach(function(item){
                        if(item.id == idx)  {
                            return k.toLowerCase();
                        }
                    });
                }
            }

        }
    }
</script>