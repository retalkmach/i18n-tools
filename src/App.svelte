<script lang="ts">
  import lightTip from "https://qidian.gtimg.com/lulu/edge/js/common/ui/LightTip.js";
  let hasProject = false;
  let fileContent: string[];
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
    fileContent = await file.text();
    //判断是否已添加同名文件
    const fileName = file.name.slice(0, file.name.indexOf(".json"));
    if (languages.includes(fileName)) {
      lightTip.error("已添加同名文件");
      return;
    }
    languages.push(fileName);
    hasProject = true;
    languages = languages;
    console.log(languages);
  }
</script>

<main>
  <nav>
    <button class="ui-button" on:click={getFile}>导入文件</button>
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
        <tr>
          <td>第1行项目1</td>
          <td>第1行项目2</td>
          <td>第1行项目3</td>
        </tr>
        <tr>
          <td>第2行项目1</td>
          <td>第3行项目2</td>
          <td>第4行项目3</td>
        </tr>
      </tbody>
    </table>
  {/if}
</main>

<style>
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/Button.css";
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/Table.css";
  @import "https://qidian.gtimg.com/lulu/edge/css/common/ui/LightTip.css";
</style>
