<template>
    <jet-form-section @submitted="updateProfileInformation">
        <template #title>
            Profile Information
        </template>

        <template #description>
            Update your account's profile information and email address.
        </template>

        <template #form>
            <!-- Profile Photo -->
            <div class="mb-3" v-if="$page.jetstream.managesProfilePhotos">
                <!-- Profile Photo File Input -->
                <input type="file" hidden
                            ref="photo"
                            @change="updatePhotoPreview">

                <jet-label for="photo" value="Photo" />

                <!-- Current Profile Photo -->
                <div class="mt-2" v-show="! photoPreview">
                    <img :src="$page.user.profile_photo_url" alt="Current Profile Photo" class="rounded-circle">
                </div>

                <!-- New Profile Photo Preview -->
                <div class="mt-2" v-show="photoPreview">
                    <img :src="photoPreview" class="rounded-circle" width="80px" height="80px">
                </div>

                <jet-secondary-button class="mt-2 mr-2" type="button" @click.native.prevent="selectNewPhoto">
                    Select A New Photo
                </jet-secondary-button>

				<jet-secondary-button type="button" class="mt-2" @click.native.prevent="deletePhoto" v-if="$page.user.profile_photo_path">
                    Remove Photo
                </jet-secondary-button>

                <jet-input-error :message="form.error('photo')" class="mt-2" />
            </div>

            <div class="w-75">
                <!-- Name -->
                <div class="mb-3">
                    <jet-label for="name" value="Name" />
                    <jet-input id="name" type="text" v-model="form.name"
                               :class="{ 'is-invalid': form.error('name') }" autocomplete="name" />
                    <jet-input-error :message="form.error('name')" />
                </div>

                <!-- Email -->
                <div class="mb-3">
                    <jet-label for="email" value="Email" />
                    <jet-input id="email" type="email" v-model="form.email"
                               :class="{ 'is-invalid': form.error('email') }" />
                    <jet-input-error :message="form.error('email')" />
            </div>
            </div>
        </template>

        <template #actions>
            <jet-action-message :on="form.recentlySuccessful" class="mr-3">
                Saved.
            </jet-action-message>

            <jet-button :class="{ 'text-white-50': form.processing }" :disabled="form.processing">
                Save
            </jet-button>
        </template>
    </jet-form-section>
</template>

<script>
    import JetButton from './../../Jetstream/Button'
    import JetFormSection from './../../Jetstream/FormSection'
    import JetInput from './../../Jetstream/Input'
    import JetInputError from './../../Jetstream/InputError'
    import JetLabel from './../../Jetstream/Label'
    import JetActionMessage from './../../Jetstream/ActionMessage'
    import JetSecondaryButton from './../../Jetstream/SecondaryButton'

    export default {
        components: {
            JetActionMessage,
            JetButton,
            JetFormSection,
            JetInput,
            JetInputError,
            JetLabel,
            JetSecondaryButton,
        },

        props: ['name', 'email'],

        data() {
            return {
                form: this.$inertia.form({
                    '_method': 'PUT',
                    name: this.name,
                    email: this.email,
                    photo: null,
                }, {
                    bag: 'updateProfileInformation',
                    resetOnSuccess: false,
                }),

                photoPreview: null,
            }
        },

        methods: {
            updateProfileInformation() {
                if (this.$refs.photo) {
                    this.form.photo = this.$refs.photo.files[0]
                }

                this.form.post(route('user-profile-information.update'), {
                    preserveScroll: true
                });
            },

            selectNewPhoto() {
                this.$refs.photo.click();
            },

            updatePhotoPreview() {
                const reader = new FileReader();

                reader.onload = (e) => {
                    this.photoPreview = e.target.result;
                };

                reader.readAsDataURL(this.$refs.photo.files[0]);
            },

            deletePhoto() {
              this.$inertia.delete(route('current-user-photo.destroy'), {
                preserveScroll: true,
              }).then(() => {
                this.photoPreview = null
              });
            },
        }
    }
</script>
