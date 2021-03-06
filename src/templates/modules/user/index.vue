<!--
Copyright 2020 DigitalOcean

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

<template>
    <div v-if="active() && app.$data.state === 'home'" class="dmt-user">
        <hr v-if="!$data.domVersion" />
        <p v-if="!$data.domVersion">
            Found user: <code>{{ user }}</code>
        </p>
        <div>
            <a class="dmt-button" @click="disableSpamWipe">Disable, spam all &amp; wipe user</a>
            <a class="dmt-button dmt-button-secondary" @click="spam">Spam all posts for user</a>
            <a class="dmt-button dmt-button-secondary" @click="disable">Disable user account</a>
            <a class="dmt-button dmt-button-secondary" @click="archive">Archive user account</a>
        </div>
        <div>
            <a class="dmt-button dmt-button-secondary" @click="wipe">Wipe user profile</a>
            <a class="dmt-button dmt-button-secondary" @click="wipeArchive">Wipe &amp; archive user</a>
            <a class="dmt-button dmt-button-secondary" @click="wipeDisable">Wipe &amp; disable user</a>
            <a class="dmt-button dmt-button-secondary" @click="archiveSpamWipe">Archive, spam all &amp; wipe user</a>
        </div>
    </div>
</template>

<script>
    const Vue = require('vue').default;
    const archiveUser = require('../../../utils/user/archiveUser');
    const disableUser = require('../../../utils/user/disableUser');
    const getUserData = require('../../../utils/user/getUserData');
    const spamAllUser = require('../../../utils/user/spamAllUser');
    const wipeUser = require('../../../utils/user/wipeUser');


    const onQuestion = () => {
        return !!(window.location.pathname.match(/\/community\/questions\/.+/));
    };
    const onUser = () => {
        return !!(window.location.pathname.match(/\/community\/users\/.+/));
    };
    const active = () => {
        return onQuestion() || onUser();
    };


    const user = active() ? (window.location.pathname.startsWith('/community/questions/') ?
        document.body.querySelector('.question-container .section-primary .postable-info-bar-container .username').textContent
        : window.location.pathname.match(/\/community\/users\/(.+)/)[1]) : null;

    let userId;
    const getUserId = async () => {
        if (userId) return userId;
        userId = (await getUserData(user)).id;
        return userId;
    };

    const running = elm => {
        elm.setAttribute('data-text-content', elm.textContent);
        elm.textContent = 'Running...';
        elm.disabled = true;
    };

    const done = elm => {
        elm.textContent = elm.getAttribute('data-text-content');
        elm.disabled = false;
        elm.removeAttribute('data-text-content');
    };

    module.exports = {
        name: 'User',
        data() {
            return {
                app: null,
                user,
            };
        },
        methods: {
            active,
            confirmation() {
                return confirm('Are you sure?') === true;
            },
            wipe(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    wipeUser(id).then(() => {
                        if (onUser()) window.location.reload();
                        done(evt.target);
                    });
                });
            },
            wipeArchive(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    wipeUser(id).then(() => {
                        archiveUser(id).then(hash => {
                            if (onUser()) window.location.href = '/community/users/' + hash;
                            done(evt.target);
                        });
                    });
                });
            },
            wipeDisable(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    wipeUser(id).then(() => {
                        disableUser(id).then(() => {
                            if (onUser()) window.location.reload();
                            done(evt.target);
                        });
                    });
                });
            },

            archive(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    archiveUser(id).then(hash => {
                        if (onUser()) window.location.href = '/community/users/' + hash;
                        done(evt.target);
                    });
                });
            },
            disable(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    disableUser(id).then(() => {
                        if (onUser()) window.location.reload();
                        done(evt.target);
                    });
                });
            },
            spam(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    spamAllUser(id).then(() => {
                        if (onUser()) window.location.reload();
                        else window.location.href = '/community/questions';
                        done(evt.target);
                    });
                });
            },

            disableSpamWipe(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    disableUser(id).then(() => {
                        spamAllUser(id).then(() => {
                            if (onUser()) window.location.reload();
                            else window.location.href = '/community/questions';
                            done(evt.target);
                        });
                    });
                });
            },
            archiveSpamWipe(evt) {
                running(evt.target);
                if (!this.confirmation()) return done(evt.target);
                getUserId().then(id => {
                    archiveUser(id).then(hash => {
                        spamAllUser(id).then(() => {
                            if (onUser()) window.location.href = '/community/users/' + hash;
                            else window.location.href = '/community/questions';
                            done(evt.target);
                        });
                    });
                });
            },
        },
        created() {
            this.$data.app = this.$parent;

            if (!this.$data.domVersion) {
                if (onUser()) {
                    // Inject a second set of buttons into the DOM
                    const DOMButtons = Vue.extend(this.constructor);
                    const instanceDOMButtons = new DOMButtons({ parent: this.$parent, data: { domVersion: true } });
                    instanceDOMButtons.$mount();
                    const elementDOMButtons = document.createElement('div');
                    elementDOMButtons.className = 'dmt Container UserProfileContainer';
                    elementDOMButtons.style.marginBottom = '1.5em';
                    elementDOMButtons.appendChild(instanceDOMButtons.$el);
                    const userContainer = document.querySelector('#users-container');
                    if (userContainer) userContainer.insertBefore(elementDOMButtons, userContainer.firstChild);
                }
            }
        },
    };
</script>
