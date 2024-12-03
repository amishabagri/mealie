 <template>
  <v-menu v-model="menu" offset-y top nudge-top="6" :close-on-content-click="false">
    <template #activator="{ on, attrs }">
      <v-btn color="accent" dark v-bind="attrs" v-on="on">
        <v-icon left>
          {{ $globals.icons.fileImage }}
        </v-icon>
        {{ $t("general.image") }}
      </v-btn>
    </template>
    <v-card width="400">
      <v-card-title class="headline flex mb-0">
        <div>
          {{ $t("recipe.recipe-image") }}
        </div>
        <AppButtonUpload
          class="ml-auto"
          url="none"
          file-name="image"
          :text-btn="false"
          :post="false"
          @uploaded="uploadImage"
        />
      </v-card-title>
      <v-card-text class="mt-n5">
        <div>
          <v-text-field v-model="url" :label="$t('general.url')" class="pt-5" clearable :messages="messages">
            <template #append-outer>
              <v-btn class="ml-2" color="primary" :loading="loading" :disabled="!slug" @click="getImageFromURL">
                {{ $t("general.get") }}
              </v-btn>
            </template>
          </v-text-field>
        </div>
      </v-card-text>
    </v-card>
  </v-menu>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, useContext } from "@nuxtjs/composition-api";
import { useUserApi } from "~/composables/api";
import { alert } from "~/composables/use-toast"; // Import the alert functions

const REFRESH_EVENT = "refresh";
const UPLOAD_EVENT = "upload";

export default defineComponent({
  props: {
    slug: {
      type: String,
      required: true,
    },
  },
  setup(props, context) {
    const state = reactive({
      url: "",
      loading: false,
      menu: false,
      messages: [],
    });


   // Function to upload the image file
   function uploadImage(fileObject: File) {
    try{
       // Check if the file has an acceptable extension
       const allowedExtensions = [".jpg", ".jpeg", ".png", ".gif", ".webp"];
      const fileExtension = fileObject.name.split(".").pop()?.toLowerCase();

      // If the file extension is not in the allowed extensions list, show an error message
      if (!fileExtension || !allowedExtensions.includes(`.${fileExtension}`)) {
        alert.error(i18n.tc("general.invalid-image-extension"));
        return;
      }

      context.emit(UPLOAD_EVENT, fileObject);
      state.menu = false;

      // Show success message for image upload
      alert.success(i18n.tc("general.image-uploaded"));
    }
      catch(error){
        alert.error(i18n.tc("general.image-upload-failed"));
      }
    }


    const api = useUserApi();
    const { i18n } = useContext();

    // Function to check if the URL is reachable
    async function isURLReachable(url: string) {
      try {
        const response = await fetch(url, { method: "HEAD" });
        return response.ok; // If status is 200, response.ok will be true
      } catch (error) {
        return false; // Network error or invalid URL
      }
    }

    // Function to get image from URL
    async function getImageFromURL() {
      state.loading = true;

      // Check if the URL is reachable it uses HTTP codes
      const isReachable = await isURLReachable(state.url);

      if (!isReachable) {
        alert.error(i18n.tc("general.invalid-url"));
        state.loading = false;
        return;
      }

      // If the URL is reachable, make the API request to update the image using the URL
      const response = await api.recipes.updateImagebyURL(props.slug, state.url);

      // Check the response from the API and show appropriate alert
      if (response) {
        alert.success(i18n.tc("general.image-uploaded"));
        context.emit(REFRESH_EVENT);
      } else {
        alert.error(i18n.tc("general.invalid-url"));
      }

      // Finalize loading state and menu visibility
      state.loading = false;
      state.menu = false;
    }

    return {
      ...toRefs(state),
      uploadImage,
      getImageFromURL,
    };
  },
});
</script>

<style lang="scss" scoped></style>
