<template>
  <Layout
    v-bind="{
      backButton: true,
      loggedInHeader: true,
      mobileTitle: mobileTitle,
      shareIdeaVisible: true,
      editIdeaVisible: true,
      onCopyShareIdeaLink: copyShareLink
    }"
    @showShareIdeaDialog="showShareIdeaDialog"
    @onEditIdea="showIdeaEditor"
  >
    <v-layout id="ideaDetailPage">
      <img class="backgroundLamp" src="~/assets/images/light_gray_lamp.png" />

      <v-flex class="profileDetails">
        <!-- Header section - only for desktop -->
        <v-layout class="sectionHeader" hidden-sm-and-down>
          <div class="headerLeftSide">
            <div class="arrowBtn">
              <v-btn text small class="leftBtn" fab>
                <v-icon> fas fa-arrow-left</v-icon>
              </v-btn>
              <v-btn text small class="leftBtn" fab>
                <v-icon>fas fa-arrow-right</v-icon>
              </v-btn>
            </div>
          </div>

          <div class="headerRightSide">
            <!-- Share Menu -->
            <v-menu class="shareMenu" offset-y>
              <template v-slot:activator="{ on }">
                <v-btn
                  text
                  icon
                  color="light-gray"
                  class="menu"
                  @click="showEmailShareDialog = true"
                  v-on="on"
                >
                  <v-icon>fas fa-envelope</v-icon>
                </v-btn>
              </template>
            </v-menu>

            <!-- Edit IDea Button-->
            <v-btn
              text
              icon
              color="gray"
              size="small"
              class="menu"
              @click="showIdeaEditor()"
            >
              <v-icon>fas fa-pen</v-icon>
            </v-btn>

            <!-- Side Settings icon -->
            <v-menu class="sideMenu" offset-y>
              <template v-slot:activator="{ on }">
                <v-btn
                  text
                  icon
                  color="gray"
                  size="small"
                  class="menu"
                  v-on="on"
                >
                  <v-icon>fas fa-ellipsis-v</v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-item>
                  <v-list-item-title>Share</v-list-item-title>
                </v-list-item>
                <v-list-item @click="copyShareLink()">
                  <v-list-item-title>Copy Direct Link</v-list-item-title>
                </v-list-item>
                <v-list-item>
                  <v-list-item-title>Report Idea</v-list-item-title>
                </v-list-item>
              </v-list>
            </v-menu>
          </div>
        </v-layout>

        <!-- Idea title -->
        <div v-if="!ideaEditorVisible" class="ideaTitle">
          <div class="ideaTitle">{{ idea.title }}</div>
        </div>
        <div v-else>
          <v-textarea v-model="idea.title" outlined label="Idea Title">
          </v-textarea>
        </div>

        <!-- Metadata -->
        <div class="metadata">
          <span>{{ user.email }}</span>
          <span class="timing">{{ idea.relativeCreatedTime }}</span>
        </div>

        <!-- Description -->
        <div v-if="!ideaEditorVisible" class="ideaDescription">
          <v-layout v-html="idea.content"> </v-layout>
        </div>
        <div v-else class="ideaEditor">
          <VueTrix
            v-model="ideaEditContents"
            class="editor"
            placeholder="Enter content"
          />
        </div>

        <!-- Tags -->
        <div v-if="!ideaEditorVisible" class="tagsContainer">
          <v-chip
            v-for="(tag, index) in ideaTags"
            :key="index"
            label
            class="tag"
            >{{ tag }}</v-chip
          >
        </div>
        <div v-else class="tagsEditor">
          <v-combobox
            v-model="chips"
            class="ideaTag"
            :items="items"
            times
            chips
            clearable
            outlined
            label="Add Tags"
            multiple
          >
            <template v-slot:selection="{ attrs, item, select, selected }">
              <v-chip
                v-bind="attrs"
                :input-value="selected"
                close
                label
                @click="select"
                @click:close="remove(item)"
              >
                <strong>{{ item }}</strong>
              </v-chip>
            </template>
          </v-combobox>
        </div>

        <!--submit and cancel btn-->
        <div v-if="ideaEditorVisible" class="buttons">
          <v-btn
            small
            color="primary"
            :loading="updatingIdea"
            @click="onSaveIdeaContent()"
            >Save</v-btn
          >
          <v-btn text small color="error" @click="ideaEditorVisible = false"
            >Cancel</v-btn
          >
        </div>

        <!-- Engagements & Next Prev -->
        <v-layout class="engagement-nextPrev" hidden-md-and-up>
          <!-- Engagement -->
          <div class="engagement">
            <div class="ups">
              <img class="lamp" src="~/assets/images/dark_gray_lamp.png" />
              <span>609</span>
            </div>
            <div class="downs">
              <img class="cmt" src="~/assets/images/comments.png" />
              <span>120</span>
            </div>
          </div>

          <!-- Mobile Only - next prev button -->
          <div class="arrowBtn">
            <v-btn text small class="leftBtn" fab>
              <v-icon> fas fa-arrow-left</v-icon>
            </v-btn>
            <v-btn text small class="leftBtn" fab>
              <v-icon>fas fa-arrow-right</v-icon>
            </v-btn>
          </div>
        </v-layout>
      </v-flex>

      <!-- Comments -->
      <v-flex class="rightSideComments">
        <v-layout class="cmtAndLike" hidden-sm-and-down>
          <div class="ups">
            <img class="lamp" src="~/assets/images/dark_gray_lamp.png" />
            <span>609</span>
          </div>
          <div class="downs">
            <img class="cmt" src="~/assets/images/comments.png" />
            <span>120</span>
          </div>
        </v-layout>

        <!-- Comment List -->
        <div v-for="i in 60" :key="i" class="commentItem">
          <div class="header">
            <div class="commentUser">Name Surname</div>
            <div class="timing">1h</div>
          </div>
          <div class="commentText">
            Excepteur sint occaecat lorem cupidatat non proident, sunt in dolor
            sit amet consecteturdfd
          </div>
        </div>
      </v-flex>

      <!-- Foter with textbox -->
      <div class="pageFooter">
        <v-text-field
          class="newCommentInput"
          flat
          solo
          label="Say something..."
          large
        ></v-text-field>

        <!-- Popip -ShowComment Dialogbox-->
        <div>
          <v-icon class="sendBtn" @click="showcommentDialog = true" v-on="on"
            >fas fa-arrow-right</v-icon
          >
        </div>

        <v-dialog
          v-model="showcommentDialog"
          content-class="commentDialog"
          persistent
          max-width="500px"
        >
          <!-- Popup Header -->
          <div class="header">
            <v-icon
              text
              class="cancelIcon"
              size="20"
              @click="showcommentDialog = false"
              >fas fa-times</v-icon
            >
          </div>

          <!-- Popup body -->
          <form>
            <div class="body">
              <div class="headlineText1">
                Oops,
              </div>
              <div class="headlineText2">
                Verify your emailor signin to post your comment.
              </div>

              <!-- Text Fields -->
              <div>
                <v-text-field
                  v-model="commentForm.Email"
                  v-validate="'required|email|max:100'"
                  class="emailInput"
                  single-line
                  flat
                  prepend-inner-icon="email"
                  :error-messages="errors.collect('email')"
                  data-vv-name="email"
                  label="Enter email"
                ></v-text-field>
              </div>

              <!-- Submit Buttons -->
              <div class="specialButton submitBtn">
                <v-btn>SEND</v-btn>
              </div>
            </div>
          </form>
        </v-dialog>
      </div>

      <!-- Popup - Share Via Email -->
      <v-dialog
        v-model="showEmailShareDialog"
        content-class="emailShareDialog"
        persistent
        max-width="400px"
      >
        <form>
          <!-- Popup Header -->

          <div class="header">
            <v-icon text class="shareIcon" size="50">fas fa-envelope</v-icon>

            <div class="headlineText">
              Share Idea by Email
            </div>
          </div>

          <!-- Text Fields -->
          <div>
            <v-text-field
              v-model="emailShareForm.name"
              v-validate="'required|max:100'"
              :error-messages="errors.collect('name')"
              data-vv-name="name"
              label="Enter your name"
              outlined
            >
            </v-text-field>

            <v-text-field
              v-model="emailShareForm.friendName"
              v-validate="'required|max:100'"
              :error-messages="errors.collect('friend name')"
              data-vv-name="friend name"
              label="Enter friend name"
              outlined
            ></v-text-field>

            <v-text-field
              v-model="emailShareForm.friendEmail"
              v-validate="'required|email|max:100'"
              :error-messages="errors.collect('email')"
              data-vv-name="email"
              label="Your Friend's email address"
              outlined
            ></v-text-field>
          </div>

          <!-- Submit Buttons -->
          <div class="btnContainer">
            <v-btn class="cancleBtn" text @click="showEmailShareDialog = false"
              >Cancel</v-btn
            >
            <v-btn class="specialButton shareBtn" @click="sendShareEmail()"
              >Share</v-btn
            >
          </div>
        </form>
      </v-dialog>

      <!-- Bottom snackbar message -->
      <v-snackbar
        v-model="snackbarVisible"
        :timeout="2000"
        :color="snackbarColor"
      >
        {{ snackbarMessage }}
        <v-btn color="white" text @click="snackbarVisible = false">
          Close
        </v-btn>
      </v-snackbar>
    </v-layout>
  </Layout>
</template>
<script>
import { graphqlOperation } from '@aws-amplify/api'
import dayjs from 'dayjs'
import relativeTime from 'dayjs/plugin/relativeTime'
import getIdea from '~/graphql/query/getIdea'
import updateIdea from '~/graphql/mutations/updateIdea'
import addComment from '~/graphql/mutations/addComment'
import Layout from '@/components/layout/Layout'
dayjs.extend(relativeTime)

export default {
  components: { Layout },
  $_veeValidate: {
    validator: 'new'
  },
  data: () => ({
    chips: ['web', 'illustration', 'graphics', 'ui', 'adobe', 'interface'],

    snackbarVisible: false,
    snackbarMessage: '',
    snackbarColor: 'success',

    showEmailShareDialog: false,
    showcommentDialog: false,
    commentForm: {
      Email: ''
    },
    dialog: false,
    emailShareForm: {
      name: '',
      friendName: '',
      friendEmail: ''
    },
    updatingIdea: false,
    ideaEditorVisible: false,
    ideaEditContents: ''
  }),
  computed: {
    mobileTitle: function() {
      console.log(this.user)
      return this.user.email.toUpperCase() + "'S IDEA"
    }
  },
  async asyncData({ app, route, store }) {
    const { data } = await app.$amplifyApi.graphql(
      graphqlOperation(getIdea, { ideaId: route.params.ideaId })
    )

    let ideaTags = [
      'web',
      'illustration',
      'graphics',
      'ui',
      'adobe',
      'interface'
    ]
    return {
      idea: data.getIdea,
      user: { email: store.state.cognito.user.attributes.email },
      ideaTags: ideaTags
    }
  },
  created() {
    this.idea.relativeCreatedTime = dayjs(this.idea.createdDate).fromNow()
  },
  methods: {
    copyShareLink() {
      this.$clipboard(window.location.href)
      this.snackbarMessage = 'Link copied'
      this.snackbarColor = 'success'
      this.snackbarVisible = true
    },
    showShareIdeaDialog() {
      this.showEmailShareDialog = true
    },
    async sendShareEmail() {
      //Validate input fields
      let result = await this.$validator.validateAll()
      if (!result) {
        return
      }

      //TODO: Send email from backend

      this.snackbarMessage = 'Email sent successfully.'
      this.showEmailShareDialog = false
      this.snackbarVisible = true
    },
    remove(item) {
      this.chips.splice(this.chips.indexOf(item), 1)
      this.chips = [...this.chips]
    },
    async addComment() {
      const idea = await this.$amplifyApi.graphql(
        graphqlOperation(addComment, { body: 'fuck tuck', ideaId: this.$route.params.ideaId, userId: this.$route.params.userId })
      )
    },
    showIdeaEditor() {
      this.ideaEditContents = this.idea.content
      this.ideaEditorVisible = true
    },
    onRemoveChip(index) {
      this.ideaTags.splice(index, 1)
    },
    async onSaveIdeaContent() {
      this.idea.content = this.ideaEditContents

      console.log('updating idea...', updateIdea)
      this.updatingIdea = true

      try {
        await this.$amplifyApi.graphql(
          graphqlOperation(updateIdea, {
            ideaId: this.$route.params.ideaId,
            content: this.idea.content,
            title: this.idea.title
          })
        )
        this.updatingIdea = false
        this.ideaEditorVisible = false

        this.snackbarMessage = 'Idea Updated'
        this.snackbarColor = 'success'
        this.snackbarVisible = true
      } catch (err) {
        this.updatingIdea = false
        this.snackbarMessage = 'Something went wrong!!'
        this.snackbarColor = 'error'
        this.snackbarVisible = true
      }
    }
    //     async deleteIdea() {
    //       await this.$amplifyApi.graphql(
    //         graphqlOperation(deleteIdea, { ideaId: this.$route.params.ideaId })
    //       )
    //     },
  }
}
</script>

<style lang="scss">
#ideaDetailPage {
  background: #ebe7ed;
  padding-bottom: 2vh;
  display: block;
  width: 100%;
  overflow-x: hidden;
  overflow: auto;
  position: relative;

  .profileDetails {
    padding: 20px;
    background: white;
    overflow: auto;

    @media #{$small-screen} {
      padding-right: 5%;
      padding-left: 5%;
    }

    .sectionHeader {
      display: flex;
      margin-bottom: 20px;

      i {
        line-height: 20px;
        font-size: 15px;
        color: #35124e;
      }

      .headerLeftSide {
        flex: 1;
      }

      .headerRightSide {
        flex: 1;
        text-align: right;

        button {
          i {
            color: #c0b7c5;
          }

          &:hover {
            i {
              color: #35124e;
            }
          }
        }
      }

      .shareMenu {
        display: inline-block;

        .arrowBtn {
          display: inline-block;
        }

        @media #{$small-screen} {
          padding-left: 10px;
          margin: 0px;
        }
      }

      .menu {
        margin: 0px;

        /* border: 1px solid red; */
      }
    }

    .ideaTitle {
      padding-bottom: 20px;
      font-size: 30px;
      font-weight: normal;
      font-style: normal;
      font-stretch: normal;
      line-height: 1.11;
      letter-spacing: normal;
      text-align: left;
      color: #18141c;
    }

    .metadata {
      padding-left: 20px;
      font-size: 12px;
      font-weight: normal;
      font-style: normal;
      font-stretch: normal;
      line-height: 1.83;
      letter-spacing: normal;
      text-align: left;
      color: #b5b5b5;

      .timing {
        padding-left: 40px;
      }
    }

    .ideaDescription {
      margin-top: 20px;

      .buttons {
        text-align: right;
      }

      @media #{$small-screen} {
        padding-top: 20px;
      }

      font-size: 15px;
      font-weight: normal;
      font-style: normal;
      font-stretch: normal;
      line-height: 1.5;
      letter-spacing: normal;
      text-align: left;
      color: #18141c;
    }

    .ideaEditor {
      margin-top: 20px;

      .editor {
        .trix-content {
          height: 170px !important;
          max-height: 200px !important;
          overflow-y: auto;
        }

        @media #{$small-screen} {
          font-size: 10px;

          .trix-content {
            height: 170px !important;
            max-height: 200px !important;
            overflow-y: auto;
          }
        }
      }
    }

    .tagsContainer {
      margin-top: 20px;

      .tag {
        margin: 0px 2px 10px 2px;
        border-radius: 6px;
        background-color: rgba(192, 183, 197);
        color: white;
      }
    }

    .tagsEditor {
      margin-top: 40px;

      .ideaTag {
        .v-chip {
          background-color: rgba(192, 183, 197);
          color: white;

          i {
            color: white;
          }
        }

        .v-input__append-inner {
          display: none;
        }
      }

      .v-icon.mdi-menu-down {
        display: none;
      }
    }

    .engagement-nextPrev {
      display: flex;
      margin-top: 15px;

      .engagement {
        flex: 1;
        padding-top: 12px;
        margin-bottom: 5px;
        padding-left: 5px;

        font-size: 18px;
        font-weight: normal;
        font-style: normal;
        font-stretch: normal;
        letter-spacing: normal;
        text-align: left;
        color: #c0b7c5;

        div {
          display: inline-block;
          margin-right: 10px;
        }

        img {
          height: 14px;
          margin-right: 5px;
        }

        img,
        .lamp {
          margin-right: 2px;
        }
      }

      .nextPrev {
        flex: 1;

        .arrowBtn {
          display: inline-block;

          i {
            font-size: 18px;
          }
        }
      }
    }
  }

  .rightSideComments {
    padding-right: 10px;
    padding-left: 10px;
    padding-top: 15px;
    padding-bottom: 50px;
    font-size: 16px;

    .cmtAndLike {
      color: #231031;
      display: flex;
      font-weight: bold;
      padding: 10px 15px 0px 15px;

      .ups {
        flex: 1;
      }
    }

    .downs {
      flex: 1;
      text-align: right;
    }

    img {
      display: inline-block;
      height: 16px;
      padding-top: 2px;
      margin-right: 5px;
    }
  }

  @media #{$small-screen} {
    padding-right: 0%;
    padding-left: 0%;
  }

  .commentTotal {
    padding: 25px 0px 10px 15px;

    font-size: 14px;
    font-weight: 600;
    font-style: normal;
    font-stretch: normal;
    line-height: 1.57;
    letter-spacing: 0.42px;
    color: #232323;
  }

  .commentItem {
    margin: 15px;
    padding: 10px;
    background-color: #ffffff;
    border-radius: 0px 7px 7px 7px;

    @media #{$small-screen} {
      border-left: 0px;
      border-right: 0px;
      border-bottom: none;
      margin-top: 0px;
    }

    .header {
      .commentUser {
        padding-bottom: 3px;
        display: inline-block;

        font-size: 12px;
        font-weight: normal;
        font-style: normal;
        font-stretch: normal;
        line-height: 1.83;
        letter-spacing: normal;
        text-align: left;
        color: #b5b5b5;
      }

      .timing {
        float: right;
        font-size: 12px;
        font-weight: normal;
        font-style: normal;
        font-stretch: normal;
        // line-height: 1.83;
        letter-spacing: normal;
        text-align: right;
        color: #c0b7c5;
      }
    }

    .commentText {
      width: 100%;
      display: block;

      @media #{$small-screen} {
        padding-top: 3px;
      }

      font-size: 14px;
      font-weight: normal;
      font-style: normal;
      font-stretch: normal;
      line-height: 1.57;
      letter-spacing: normal;
      text-align: left;
      color: #827c85;
    }
  }
}

.pageFooter {
  position: fixed;
  width: 100%;
  bottom: 0;
  left: 0;
  background: white;
  z-index: 100;

  height: 68px;
  padding-top: 10px;
  padding-left: 10px;
  -webkit-backdrop-filter: blur(30px);
  backdrop-filter: blur(30px);
  box-shadow: 0 -10px 10px 0 rgba(228, 228, 228, 0.73);
  background-color: #ffffff;

  .newCommentInput {
    .v-input__slot {
      margin-bottom: 0px;
    }
  }

  .sendBtn {
    position: absolute;
    top: 24px;
    right: 19px;
    font-size: 21px;
    color: #d4bb10;
  }
}

.backgroundLamp {
  position: absolute;
  left: -10%;
  top: 25vh;
  width: 20%;
  z-index: 0;

  @media #{$small-screen} {
    left: 60%;
    top: 15vh;
    width: 80%;
    z-index: 0;
  }
}

.commentDialog {
  padding: 15px 20px 20px 20px;
  background: white;
  width: 100%;

  .header {
    .cancelIcon {
      float: right;
    }
  }

  .body {
    padding: 20px 20px 20px 20px;

    .headlineText1 {
      padding-top: 20px;
      font-size: 25px;
      text-align: center;
    }

    .headlineText2 {
      padding-top: 10px;
      padding-bottom: 10px;
      text-align: center;
      font-size: 13px;
      color: rgba(192, 183, 197);
    }

    .emailInput {
      padding-top: 20px;
      margin: 0px;

      .v-icon {
        font-size: 18px;
      }
    }

    .submitBtn {
      padding-top: 20px;
      text-align: center;

      button {
        width: 100%;
      }
    }
  }
}

.emailShareDialog {
  padding: 40px 20px 20px 20px;
  background: white;

  .header {
    text-align: center;
    margin-bottom: 20px;

    .headlineText {
      margin-top: 10px;
      font-size: 25px;
      font-family: Quatro;
    }

    .cancelIcon {
      border: 1px solid red;
      float: right;
      padding-bottom: 10px;
    }
  }

  .shareIcon {
    color: rgba(255, 185, 45);
  }

  .btnContainer {
    margin-top: 20px;
    margin-bottom: 20px;
    text-align: right;

    .shareBtn {
      width: 170px;
    }
  }
}
</style>