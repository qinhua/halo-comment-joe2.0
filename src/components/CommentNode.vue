<template>
  <div class="comment-wrp">
    <li
      :id="'comment-' + comment.id"
      class="comment"
      :class="commentClass"
      itemtype="http://schema.org/Comment"
      itemprop="comment"
    >
      <div class="contents">
        <div class="main shadow">
          <div class="profile">
            <a
              :class="{ disabled: invalidUrl(comment.authorUrl) }"
              :href="comment.authorUrl"
              rel="nofollow noopener noreferrer"
              target="_blank"
            >
              <img
                v-lazy="currentAvatar(comment)"
                class="avatar"
                height="80"
                width="80"
                :alt="comment.author"
                @error="handleAvatarError"
              />
            </a>
          </div>
          <div class="commentinfo">
            <section class="commeta">
              <div class="left">
                <h4 class="author">
                  <a
                    :class="{ disabled: invalidUrl(comment.authorUrl) }"
                    :href="comment.authorUrl"
                    rel="nofollow noopener noreferrer"
                    target="_blank"
                  >
                    <img
                      :alt="comment.author"
                      v-lazy="currentAvatar(comment)"
                      class="avatar"
                      height="24"
                      width="24"
                      @error="handleAvatarError"
                    />
                    <span
                      v-if="comment.isAdmin"
                      class="bb-comment isauthor"
                      title="博主"
                      >博主</span
                    >
                    {{ comment.author }}
                  </a>
                </h4>
              </div>
              <a
                class="comment-reply-link"
                :style="editing ? 'display:block;' : ''"
                href="javascript:;"
                @click="handleReplyClick"
                >回复</a
              >
              <div class="right">
                <div class="info">
                  <time
                    class="comment-time"
                    itemprop="datePublished"
                    :datetime="comment.createTime"
                    >发布于 {{ createTimeAgo }}
                  </time>
                  <span
                    v-if="configs.showUserAgent"
                    class="useragent-info"
                    v-html="compileUserAgent"
                  ></span>
                </div>
              </div>
            </section>
          </div>
          <div class="body markdown-body">
            <!-- 将所有的评论内容约束为一段 -->
            <div class="markdown-content" v-html="compileContent"></div>
          </div>
        </div>
      </div>
      <ul v-if="comment.children" class="children">
        <template v-for="(children, index) in comment.children">
          <CommentNode
            :isChild="true"
            :targetId="targetId"
            :target="target"
            :comment="children"
            :options="options"
            :configs="configs"
            :key="index"
            :depth="selfAddDepth"
            :parent="comment"
          />
        </template>
      </ul>
    </li>
    <CommentEditor
      :targetId="targetId"
      :target="target"
      :replyComment="comment"
      :options="options"
      :configs="configs"
    />
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */
import "./index";
import { timeAgo, return2Br } from "@/utils/util";
import ua from "ua-parser-js";
import marked from "j-marked";
import { renderedEmojiHtml } from "@/utils/emojiutil";
import CommentEditor from "./CommentEditor.vue";
import globals from "@/utils/globals.js";

export default {
  name: "CommentNode",
  components: {
    CommentEditor,
  },
  props: {
    parent: {
      type: Object,
      required: false,
      default: undefined,
    },
    depth: {
      type: Number,
      required: false,
      default: 1,
    },
    isChild: {
      type: Boolean,
      required: false,
      default: false,
    },
    targetId: {
      type: Number,
      required: false,
      default: 0,
    },
    target: {
      type: String,
      required: false,
      default: "posts",
      validator: function (value) {
        return ["posts", "journals", "sheets"].includes(value);
      },
    },
    comment: {
      type: Object,
      required: false,
      default: () => {},
    },
    options: {
      type: Object,
      required: false,
      default: () => {},
    },
    configs: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      editing: false,
      globalData: globals,
      error_img: `${process.env.BASE_URL}assets/svg/img_error.svg`,
      unknow_ua: `${process.env.BASE_URL}assets/ua/unknow.svg`,
      empty_img:
        "data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==",
    };
  },
  created() {
    const renderer = {
      // eslint-disable-next-line no-unused-vars
      image(href, title) {
        return `<a data-fancybox target="_blank" rel="noopener noreferrer nofollow" href="${href}"><img src="${href}" class="lazyload comment_inline_img" onerror="this.src='${this.error_img}'"></a>`;
      },
      link(href, title, text) {
        return `<a href="${href}" title="${text}" target="_blank" rel="noopener noreferrer nofollow">${text}</a>`;
      },
    };
    marked.use({ renderer });
  },
  computed: {
    avatar() {
      // if (this.comment.avatar) {
      //   return this.comment.avatar;
      // } else {
      // !!优先从主题配置取，取不到才从后台配置取
      const gravatarSource =
        this.configs.gravatarSource ||
        this.options.gravatar_source ||
        this.configs.gravatarSourceDefault;

      return `${gravatarSource}/${this.comment.gravatarMd5}?s=256&d=${
        this.options.comment_gravatar_default || "mm"
      }`;
      // }
    },
    compileContent() {
      var at = "";
      if (this.parent != undefined) {
        // at =
        //   '<a href="' +
        //   this.parent.authorUrl +
        //   '" class="comment-at" rel="noopener noreferrer nofollow"> @' +
        //   this.parent.author +
        //   " </a>";
        at =
          '<a href="javascript:;" class="comment-at"> @' +
          this.parent.author +
          " </a>";
      }
      // 获取转换后的marked
      const markedHtml = marked(at + this.comment.content);
      // 处理其中的表情包
      const emoji = renderedEmojiHtml(markedHtml);
      // 将回车转换为br
      return return2Br(emoji);
    },
    createTimeAgo() {
      return timeAgo(this.comment.createTime);
    },
    compileUserAgent() {
      if (!this.comment.userAgent) return "";
      var parser = new ua();
      parser.setUA(this.comment.userAgent);
      var result = parser.getResult();

      if (!result.browser.name) return "";
      var browserImg = `${process.env.BASE_URL}assets/ua/${decodeURIComponent(
        result.browser.name.toLowerCase()
      )}.svg`;
      var uaImg = "";

      switch (result.os.name) {
        case "Windows":
          switch (result.os.version) {
            case "7":
            case "8":
            case "10":
              uaImg = `${
                process.env.BASE_URL
              }assets/ua/windows_win${decodeURIComponent(
                result.os.version
              )}.svg`;
              break;
            case "":
              uaImg = `${
                process.env.BASE_URL
              }assets/ua/windows_${decodeURIComponent(result.os.version)}.svg`;
              break;
            default:
              uaImg = `${process.env.BASE_URL}assets/ua/windows.svg`;
              break;
          }
          break;
        default:
          uaImg = `${process.env.BASE_URL}assets/ua/${decodeURIComponent(
            result.os.name
          )
            .replace(/\s+/g, "")
            .toLowerCase()}.svg`;
          break;
      }

      return `（<img src="${browserImg}" onerror="this.src='${this.unknow_ua}'" alt="ua-browser"/>  ${result.browser.name} ${result.browser.version} <img src="${uaImg}" onerror="this.src='${this.unknow_ua}'" alt="ua-os"/> ${result.os.name} ${result.os.version}）`;
    },
    selfAddDepth() {
      return this.depth + 1;
    },
    commentClass() {
      return "depth-" + this.depth + " comment-" + this.comment.id;
    },
  },
  methods: {
    currentAvatar(data) {
      return data.isAdmin ? this.configs.bloggerAvatar : this.avatar;
    },
    invalidUrl(url) {
      return !/^http(s)?:\/\//.test(url);
    },
    handleReplyClick(e) {
      e.stopPropagation();
      // 设置状态为回复状态
      this.globalData.replyId = this.comment.id;
    },
    handleAvatarError(e) {
      const img = e.target || e.srcElement;
      img.src = this.configs.avatarError;
      img.onerror = null;
    },
  },
};
</script>
