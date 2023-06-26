<template>
  <div class="flex h-1/6">
    <textarea class="w-10/12 md:w-11/12 h-24 border border-gray-400 " placeholder="原始内容" v-model="text" />
    <div class="w-2/12 md:w-1/12">
      <button class="h-8 mt-16 w-full border border-gray-400 bg-blue-500 text-white" @click="send()">生成脑图</button>
    </div>
  </div>
  <div class="w-full md:flex h-5/6">
    <textarea class="w-full md:w-5/12 border border-gray-400 resize" placeholder="markdown格式" v-model="value" />
    <svg class="w-full md:w-7/12 border border-gray-400" ref="svgRef" />
  </div>
</template>

<script>
import { ref, onMounted, onUpdated } from 'vue';
import { Transformer } from 'markmap-lib';
import { Markmap } from 'markmap-view/dist/index.esm';

const transformer = new Transformer();

const initText = `markmap: markdown 制作脑图
1. beautiful 颜值高
2. useful  有价值
3. easy  易上手
4. interactive 可交互
`;
const initValue = `# markmap: markdown 制作脑图

- beautiful 颜值高
- useful  有价值
- easy  易上手
- interactive 可交互
`;

const decoder = new TextDecoder("utf-8");

export default {
  name: 'App',
  data() {
    return {
      value: initValue,
      text: initText,
    };
  },
  watch: {
    value: 'update',
  },
  methods: {
    update() {
      const { root } = transformer.transform(this.value);
      this.mm.setData(root);
      this.mm.fit();
    },
    async send() {
      try {
        const messageList = [
          {
            role: "system",
            content: `你是制作兼容脑图的Markdown格式的专家,只返回markdown格式内容`,
          },
          {
            role: "user",
            content:
              `请根据内容生成兼容脑图的Markdown格式，包含一级内容，和分级目录，下面是内容：\n` + this.text,
          },
        ]
        const { body, status } = await fetch("https://www.13042332817.top/message", {
          method: "post",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            message: messageList,
          }),

        });
        if (body) {
          const reader = body.getReader();
          let partialLine = "";
          this.value = "";
          let index = 0;
          while (true) {
            const { value, done } = await reader.read();
            if (done) break;

            const decodedText = decoder.decode(value, { stream: true });

            if (status !== 200) {
              const json = JSON.parse(decodedText); // start with "data: "
              const content = json.error.message ? json.error.message : decodedText;
              appendLastMessageContent(content);
              return;
            }
            partialLine = partialLine + decodedText

            if (index > 10 && Math.random() > 0.8) {
              this.value = partialLine
            }
            index++;
          }
          this.value = partialLine
        }
      } catch (error) {
        console.log(error)
      } finally {

      }
    },
  },
  mounted() {
    this.mm = Markmap.create(this.$refs.svgRef);
    setTimeout(() => {
      this.update();
    }, 200)
  },
};
</script>
