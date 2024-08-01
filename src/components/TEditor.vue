<template>
  <editor :id="editorid" v-model="tinycontent" :disabled="props.disabled" :init="tinyinit" />
</template>

<script setup lang="ts">
import Editor from '@tinymce/tinymce-vue';
import { computed, ref } from 'vue';

import { TINY_FONT_FAMILY_FORMATS, TINY_FONT_SIZE_FORMATS, TINY_PLUGINS, TINY_TOOLBAR } from './constants';

const props = defineProps({
  modelValue: {
    type: String,
    required: true,
    default: '',
  },
  editorid: {
    type: String,
    default: () => 'tiny'
  },
  plugins: {
    type: [String, Array],
    default: TINY_PLUGINS,
  },
  toolbar: {
    type: [String, Array],
    default: TINY_TOOLBAR,
  },
  toolbarMode: {
    type: String,
    default: 'sliding',
    // default: 'wrap',
  },
  fontFamilyFormats: {
    type: String,
    default: TINY_FONT_FAMILY_FORMATS,
  },
  fontSizeFormats: {
    type: String,
    default: TINY_FONT_SIZE_FORMATS,
  },
  // extra
  disabled: {
    type: Boolean,
    default: false
  }
});

const upload_handler = (blobInfo, progress) => new Promise((resolve, reject) => {
  const xhr = new XMLHttpRequest();
  xhr.withCredentials = false;
  xhr.open('POST', 'postAcceptor.php');

  xhr.upload.onprogress = (e) => {
    progress(e.loaded / e.total * 100);
  };

  xhr.onload = () => {
    if (xhr.status === 403) {
      reject({ message: 'HTTP Error: ' + xhr.status, remove: true });
      return;
    }

    if (xhr.status < 200 || xhr.status >= 300) {
      reject('HTTP Error: ' + xhr.status);
      return;
    }

    const json = JSON.parse(xhr.responseText);

    if (!json || typeof json.location != 'string') {
      reject('Invalid JSON: ' + xhr.responseText);
      return;
    }

    resolve(json.location);
  };

  xhr.onerror = () => {
    reject('Image upload failed due to a XHR Transport error. Code: ' + xhr.status);
  };

  const formData = new FormData();
  formData.append('file', blobInfo.blob(), blobInfo.filename());

  xhr.send(formData);
});

const tinyinit = ref({
  selector: `#{props.editorid}`,
  height: '100%',
  resize: true, // 不允许用户调整大小
  language: 'zh_CN', // 汉化
  branding: false, // 隐藏tinymce右下角水印
  convert_urls: false, // 不自动转换链接地址
  plugins: props.plugins, // 插件
  contextmenu: '', // 上下文菜单
  menubar: 'edit view insert format tools table help', // 菜单栏
  toolbar_mode: props.toolbarMode, // 工具栏多行显示样式
  toolbar: props.toolbar, // 工具栏
  font_family_formats: props.fontFamilyFormats, // 字体选择
  font_size_formats: props.fontSizeFormats, // 字号选择
  image_advtab: true,
  image_caption: true,
  importcss_append: true,
  file_picker_types: 'file image media',
  image_uploadtab: true,
  // images_upload_url: 'postAcceptor.php',
  // images_upload_handler: upload_handler,
  file_picker_callback: (callback, value, meta) => {
    /* Provide file and text for the link dialog */
    if (meta.filetype === 'file') {
      // callback('https://www.google.com/logos/google.jpg', { text: 'My text' });
    }

    /* Provide image and alt text for the image dialog */
    if (meta.filetype === 'image') {
      // callback('https://www.google.com/logos/google.jpg', { alt: 'My alt text' });
      const input = document.createElement('input');
      input.setAttribute('type', 'file');
      input.setAttribute('accept', 'image/*');

      input.addEventListener('change', (e) => {
        const file = e.target.files[0];

        const reader = new FileReader();
        reader.addEventListener('load', () => {
          /*
            Note: Now we need to register the blob in TinyMCEs image blob
            registry. In the next release this part hopefully won't be
            necessary, as we are looking to handle it internally.
          */
          const id = 'blobid' + (new Date()).getTime();
          const blobCache = tinymce.activeEditor.editorUpload.blobCache;
          const base64 = reader.result.split(',')[1];
          const blobInfo = blobCache.create(id, file, base64);
          blobCache.add(blobInfo);

          /* call the callback and populate the Title field with the file name */
          callback(blobInfo.blobUri(), { title: file.name });
        });
        reader.readAsDataURL(file);
      });

      input.click();
    }

    /* Provide alternative source and posted for the media dialog */
    if (meta.filetype === 'media') {
      // callback('movie.mp4', { source2: 'alt.ogg', poster: 'https://www.google.com/logos/google.jpg' });
    }
  },
});

const emit = defineEmits(['update:modelValue']);

// 双向绑定
const tinycontent = computed({
  get(): string {
    return props.modelValue;
  },
  set(value) {
    emit('update:modelValue', value);
  },
});
</script>

<style scoped></style>
