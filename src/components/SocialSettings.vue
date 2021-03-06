<template>
    <v-card flat class="pt-6 pb-6">
        <v-row class="align-center">
            <v-form @submit.prevent="setTestingTask" class="container" v-model="testingActivityValid">
                <v-row class="align-center">
                    <v-col cols="2">
                        Тестовое задание:
                    </v-col>
                    <v-col cols="3">
                        <v-select v-model="testingActivity"
                                  item-text="title"
                                  item-value="id"
                                  :items="activities"></v-select>
                    </v-col>
                    <v-col cols="3">
                        <v-text-field :rules="testingDataRules"
                                      v-model="testingActivityData"
                                      label="Ссылка"></v-text-field>
                    </v-col>
                    <v-col>
                        <v-btn :disabled="!testingActivityValid" color="green accent-4"
                               type="submit" :dark="testingActivityValid" >Обновить</v-btn>
                    </v-col>
                </v-row>
            </v-form>
        </v-row>

        <!--Таблица с данными-->
        <v-data-table
                class="mt-3"
                :headers="headers"
                :items="activities">
            <template v-slot:item.human_price="props">
                <v-edit-dialog
                        :return-value.sync="props.item.human_price"
                        @save="save(props.item, 'human_price')"
                        @open="open(props.item.human_price)"> {{ props.item.human_price }}
                    <template v-slot:input>
                        <v-text-field
                                type="number"
                                v-model="props.item.human_price"
                                label="Изменить"
                                single-line
                        ></v-text-field>
                    </template>
                </v-edit-dialog>
            </template>
            <template v-slot:item.bot_price="props">
                <v-edit-dialog
                        :return-value.sync="props.item.bot_price"
                        @save="save(props.item, 'bot_price')"
                        @open="open(props.item.bot_price)"> {{ props.item.bot_price }}
                    <template v-slot:input>
                        <v-text-field
                                type="number"
                                v-model="props.item.bot_price"
                                label="Изменить"
                                single-line
                        ></v-text-field>
                    </template>
                </v-edit-dialog>
            </template>
            <template v-slot:item.limit="props">
                <v-edit-dialog
                        :return-value.sync="props.item.limit"
                        @save="save(props.item, 'limit')"
                        @open="open(props.item.limit)"> {{ props.item.limit }}
                    <template v-slot:input>
                        <v-text-field
                                type="number"
                                v-model="props.item.limit"
                                label="Изменить"
                                single-line
                        ></v-text-field>
                    </template>
                </v-edit-dialog>
            </template>

            <template v-slot:item.human_enable="{ item }">
                <v-simple-checkbox @click="open(item.human_enable)"
                                   @input="save(item, 'human_enable')"
                                   v-model="item.human_enable"></v-simple-checkbox>
            </template>
            <template v-slot:item.bot_enable="{ item }">
                <v-simple-checkbox @click="open(item.bot_enable)"
                                   @input="save(item, 'human_enable')"
                                   v-model="item.bot_enable"></v-simple-checkbox>
            </template>
        </v-data-table>

        <!--Всплывашка-->
        <v-snackbar v-model="snack" :timeout="3000" :color="snackColor">
            <div v-if="snackColor == 'error'">
                <span class="d-block" v-for="err in errors">{{ err }}</span>
            </div>
            <span v-else>{{ snackText }}</span>
            <v-btn text @click="snack = false">Закрыть</v-btn>
        </v-snackbar>
    </v-card>
</template>

<script>
    import axios from "axios"

    export default {
        props: {
            social: {
                type: Object,
                required: true
            }
        },
        name: "SocialSettings",
        data() {
            return {
                testingActivityData: "",
                testingActivity: null,
                testingActivityValid: true,
                snack: false,
                errors: [],
                snackColor: '',
                snackText: '',
                pagination: {},
                fetchedSettings: [],

                headers: [
                    {
                        text: 'Целевое действие',
                        align: 'start',
                        value: 'title',
                    },
                    { text: 'Цена за единицу', value: 'human_price' },
                    { text: 'Цена (боты)', value: 'bot_price' },
                    { text: 'Кол-во заданий в час', value: 'limit' },
                    { text: 'Люди', value: 'human_enable' },
                    { text: 'Боты', value: 'bot_enable' },
                ],
                // Для возвращения значения в случае ошибки
                lastValue: null,
                // Правила для ссылки
                testingDataRules: [
                    v => !!v || "Заполните поле"
                ]
            }
        },
        computed: {
            activities: {
                get() {
                    let res = Array.from(this.social.cheatingTypes);

                    for(let i in res) {
                        res[i] = Object.assign({}, this.fetchedSettings[i], res[i])
                    }

                    return res
                },
                set(val) {
                }
            }
        },
        mounted() {
            axios.post(this.$store.getters.getAPIurl + "/social/list", {
                type: this.social.id
            }).then((res)=>{
                this.fetchedSettings = res.data.data.settings;

                for(let s of this.fetchedSettings) {
                    s.human_enable = !!s.human_enable;
                    s.bot_enable = !!s.bot_enable;
                }
            });

            axios.post(this.$store.getters.getAPIurl + "/social/tasklist", {
                type: this.social.id
            }).then((res)=>{
                if(res.data.data.settings.length > 0) {
                    this.testingActivity = res.data.data.settings[0].action_id;
                    this.testingActivityData = res.data.data.settings[0].task_url;
                }
            });

            this.testingActivity = this.social.cheatingTypes[0].id;
        },
        methods: {
            setTestingTask() {
                let params = {
                    social_type: this.social.id,
                    order_type: this.testingActivity,
                    url: this.testingActivityData,
                };

                axios.post(this.$store.getters.getAPIurl+"/social/task", params).then((res)=>{
                    this.snack = true
                    this.snackColor = 'success'
                    this.snackText = 'Изменения сохранены'
                }).catch((err)=>{
                    this.parseErrors(err.response.data.errors);
                    this.snack = true;
                    this.snackColor = 'error';
                });
            },

            save (item, field) {
                let action = {
                    order_type: item.action_id,
                    social_type: item.social_id,
                    human_price: item.human_price,
                    bot_price: item.bot_price,
                    human_enable: item.human_enable,
                    bot_enable: item.bot_enable,
                    limit: item.limit
                };

                axios.post(this.$store.getters.getAPIurl + "/social/edit", action).then(()=>{
                    this.snack = true
                    this.snackColor = 'success'
                    this.snackText = 'Изменения сохранены'
                }).catch((err)=>{
                    item[field] = this.lastValue;
                    this.parseErrors(err.response.data.errors)
                    this.snack = true;
                    this.snackColor = 'error';
                });
            },

            parseErrors(errors) {
                this.errors = []

                for(let errArr in errors) {
                    for(let e of errors[errArr]) {
                        this.errors.push(e);
                    }
                }
            },

            open (val) {
                this.lastValue = val;
            },
        },
    }
</script>

<style scoped>

</style>