<template>
    <NavigationBar />
    <v-container>
        <v-col>
            <v-layout v-for="(doc) in docs" :key="doc.id">
                <v-card class="mx-auto ma-2 pa-2" hover @click="expand(doc.id)" width="1200px">
                    <v-card-item class="text-h5">
                        <v-card-title>ID: {{ doc.id }} {{ doc.title }}</v-card-title>
                        <v-row>
                            <v-col>
                                <v-card-text>
                                    Last Edit time: {{ doc.date }} {{ doc.time }}
                                </v-card-text>
                            </v-col>
                            <v-col>
                                <v-card-text>
                                    Approver: {{ doc.approver }}
                                </v-card-text>
                            </v-col>
                            <v-col>
                                <v-card-text>
                                    Status: {{ doc.status }}
                                </v-card-text>
                            </v-col>
                        </v-row>
                        <template v-slot:append>
                            <v-btn color="primary" text="Edit" variant="text" @click="test"></v-btn>
                        </template>
                    </v-card-item>
                    <v-expand-transition>
                        <div v-if="doc.expand">
                            <v-list lines="two">
                                <v-list-subheader>File History</v-list-subheader>
                                <v-list-item-group>
                                    <v-layout v-for="(history) in doc.history" :key="history.id">
                                        <v-list-item>
                                            <v-list-item-title>{{ history.status }} @ {{ history.date }}
                                                {{ history.time }}</v-list-item-title>
                                            <v-list-item-subtitle>Comment: {{ history.comment }}</v-list-item-subtitle>
                                        </v-list-item>
                                    </v-layout>
                                </v-list-item-group>
                            </v-list>
                        </div>
                    </v-expand-transition>
                </v-card>
            </v-layout>
        </v-col>
    </v-container>
</template>

<script>
import { getDocList, getDocHistory } from "@/api/docApi";
import NavigationBar from "@/components/NavigationBar.vue";
export default {
    name: "DocHistory",
    components: {
        NavigationBar,
    },
    data() {
        return {
            docs: [],
        };
    },
    async created() {
        const docList = await getDocList(this.$store.state.login.id);
        if (docList.documents === null) {
            return;
        }
        try {
            this.docs = docList.documents.map((doc) => {
                return {
                    id: doc["id"],
                    approver: doc["approver"],
                    title: doc["title"] || "untitled",
                    level: doc["status"] === "EDIT" ? 1 : doc["status"] === "VERIFY" ? 2 : doc["status"] === "REJECT" ? 3 : 0,
                    status: doc["status"],
                    date: new Date(new Date(doc["updated_at"]).getTime()).toLocaleDateString(),
                    time: new Date(new Date(doc["updated_at"]).getTime()).toLocaleTimeString([], { hour12: false }),
                    history: [],
                    expand: false,
                };
            });
            this.docs.sort((a, b) => {
                return new Date(b.date + " " + b.time) - new Date(a.date + " " + a.time);
            });
        } catch (err) {
            console.log(err);
        }
    },
    methods: {
        expand(id) {
            this.docs = this.docs.map((doc) => {
                if (doc.id === id) {
                    try {
                        const filehistory = getDocHistory(id, this.$store.state.login.id);
                        filehistory.then((res) => {
                            if (res.histories !== null) {
                                doc.history = res.histories.map((history) => {
                                    return {
                                        id: history["id"],
                                        status: history["status"],
                                        date: new Date(new Date(history["updated_at"]).getTime()).toLocaleDateString(),
                                        time: new Date(new Date(history["updated_at"]).getTime()).toLocaleTimeString([], { hour12: false }),
                                        comment: history["comment"],
                                    };
                                });
                            }
                        });
                    } catch (error) {
                        console.log(error);
                    }
                    doc.expand = !doc.expand;
                } else {
                    doc.expand = false;
                }
                return doc;
            });
        },
        async test() {
            console.log("test");
        },
    },
};
</script>