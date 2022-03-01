<script lang="ts">
  import lightTip from "https://qidian.gtimg.com/lulu/edge/js/common/ui/LightTip.js";
  import { saveAs } from "file-saver";
  import * as JSZip from "jszip";
  let hasProject = false;
  let translateContents: {} = {};
  let originWords: string[] = [];
  let languages: string[] = [];

  async function getFile() {
    let config = {
      types: [
        { description: "i18n json file", accept: { "files/*": [".json"] } },
      ],
      excludeAcceptAllOption: true,
      multiple: false,
    };
    let [fileHandle] = await window.showOpenFilePicker(config);
    const file: File = await fileHandle.getFile();
    let fileContent = await file.text();
    //判断是否已添加同名文件
    const fileName = file.name.slice(0, file.name.indexOf(".json"));
    if (languages.includes(fileName)) {
      lightTip.error("已添加同名文件");
      return;
    }
    translateContents[fileName] = JSON.parse(fileContent);
    languages.push(fileName);
    updateLanguage(translateContents[fileName]);
    hasProject = true;
    languages = languages;
  }
  function updateLanguage(data: object) {
    let dataKeys = Object.keys(data);
    dataKeys.map((key) => {
      if (!originWords.includes(key)) {
        originWords[originWords.length] = key;
      }
    });
  }
  function updateTranslateContent(e) {
    let target = e.target;
    let lang = target.dataset.lang;
    let word = target.dataset.word;
    translateContents[lang][word] = target.value;
  }
  async function exportFile() {
    let fileContent = JSON.stringify(translateContents);
    let zipFile: JSZip = new JSZip();
    for (let [key, value] of Object.entries(translateContents)) {
      zipFile.file(`${key}.json`, new Blob([JSON.stringify(value)]));
    }
    zipFile.generateAsync({ type: "blob" }).then((res) => {
      saveAs(res, "locales.zip");
    });
  }
  function renameOriginWord(e) {
    let oldKeyName = originWords[e.target.dataset.index];
    //判断是否存在相同键名
    if (originWords.includes(e.target.value)) {
      lightTip.error("存在相同键名");
      e.target.value = oldKeyName;
      return;
    }
    originWords[e.target.dataset.index] = e.target.value;
    let newKeyName = e.target.value;
    for (let language of languages) {
      translateContents[language][newKeyName] =
        translateContents[language][oldKeyName];
      delete translateContents[language][oldKeyName];
    }
  }
  function addOriginWord() {
    console.log("test");
    if (originWords.includes("needtranslate")) {
      lightTip.error("已存在待翻译键名");
      return;
    }
    originWords[originWords.length] = "needtranslate";
    for (let language of languages) {
      translateContents[language]["needtranslate"] = "";
    }
  }
</script>

<main>
  <nav>
    <button class="ui-button" on:click={getFile}>导入文件</button>
    {#if hasProject}
      <button class="ui-button" on:click={exportFile}>导出文件</button>
      <button class="ui-button" on:click={addOriginWord}>添加新行</button>
    {/if}
  </nav>
  {#if hasProject}
    <table class="ui-table">
      <thead>
        <tr>
          <th>源单词</th>
          {#each languages as language, index}
            <th>{languages[index]}</th>
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each originWords as word, index}
          <tr>
            <td>
              <input
                class="ui-input"
                type="text"
                value={word}
                data-index={index}
                on:change={renameOriginWord}
              /></td
            >
            {#each languages as language}
              <td
                ><input
                  class="ui-input"
                  type="text"
                  value={translateContents[language][word] ?? ""}
                  data-lang={language}
                  data-word={word}
                  on:change={updateLanguage}
                /></td
              >
            {/each}
          </tr>
        {/each}
      </tbody>
    </table>
  {/if}
</main>

<style>
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/Button.css";
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/Input.css";
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/LightTip.css";
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/Table.css";
</style>
