<template>
  <div id="app">
    <button @click="convert">Convert</button>
    <button @click="toRequest">【API】Request</button>
    <button @click="toResponce">【API】Responce</button>
    <button @click="toToDomainModel">【API】DomainModel</button>
    <br />
    <textarea v-model="input" name="example1" cols="50" rows="30"> </textarea>
    <textarea v-model="template" name="example2" cols="50" rows="30"></textarea>
    <h2>output</h2>

    <button @click="copy">copy to clipboard</button>
    <pre
      >{{ code }}
        </pre
    >
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue';

export default {
  name: 'App',
  components: {
    HelloWorld,
  },
  data() {
    return {
      input: ``,
      template: '##1 ##2 ##3 ##1 ##2 ##3',
      code: '',
      responceCode: ``,
    };
  },
  methods: {
    convert: function () {
      let rows = this.input.split('\n');
      rows = rows.filter((x) => x != '');

      let ret = '';
      rows.forEach((row, i_row) => {
        let items = row.split('\t');
        items = items.filter((x) => x != '');
        let loopVal = this.template.replace(/##1/g, items[0]);
        loopVal = loopVal.replace(/##2/g, items[1]);
        loopVal = loopVal.replace(/##3/g, items[2]);
        ret += loopVal + '\n';
      });

      //this.toUpperFirstLetter(items[0]) +
      this.code = ret;
      //this.code = this.toCSfield(this.input);
      //this.template = this.
      //this.responceCode = this.toCSfield(this.input);
    },
    toRequest: function () {
      this.code = this.toCSfield(this.input);
      this.responceCode = this.toCSfield(this.input);
    },
    toResponce: function () {
      this.code = this.toCSfield(this.input);
      this.code += '\n';
      this.code += this.toConstuctor(this.input);
    },
    toToDomainModel: function () {
      let ret = '/// <summary>\n';
      ret += '/// ViewModel→DomainModelの変換\n';
      ret += '/// </summary>\n';
      ret += 'public @ISearchOption ToDomainModel(){\n';
      ret += '\tvar searchOption = new @SearchOption();\n\n';
      const fieldNames = this.getFieldNames(this.input);
      fieldNames.forEach((fieldName) => {
        ret += '\tsearchOption.' + fieldName + ' = ' + fieldName + ';\n';
      });

      ret += '}';
      this.code = ret;
    },
    toCSfield: function (ipt) {
      let rows = ipt.split('\n');
      console.log(rows);
      rows = rows.filter((x) => x != '');
      //変換したい型が増えたらここに追加
      const types = { string: 'string', 'string($date-time)': 'DateTime' };
      let ret = '';
      rows.forEach((row, i_row) => {
        let items = row.split('\t');
        items = items.filter((x) => x != '');
        let typeCsvStr = items[1];
        const typeStr = typeCsvStr in types ? types[typeCsvStr] : typeCsvStr;
        ret += '/// <summary>\n';
        ret += '/// ' + items[2] + '\n';
        ret += '/// </summary>\n';
        ret +=
          'public ' +
          typeStr +
          ' ' +
          this.toUpperFirstLetter(items[0]) +
          ' {get; set;}\n\n';
      });
      return ret;
    },

    toConstuctor: function (ipt) {
      let ret = '/// <summary>\n';
      ret += '/// DomainModel→ViewModelの変換\n';
      ret += '/// </summary>\n';
      ret += 'internal @ViewModel(@DomainModel model){\n';

      const fieldNames = this.getFieldNames(ipt);

      fieldNames.forEach((fieldName) => {
        ret += '\t' + fieldName + ' = model.' + fieldName + ';\n';
      });

      ret += '}\n';
      return ret;
    },

    getFieldNames: function (ipt) {
      let rows = ipt.split('\n');
      rows = rows.filter((x) => x != '');
      let ret = [];
      rows.forEach((row, i_row) => {
        let items = row.split('\t');
        items = items.filter((x) => x != '');
        ret.push(items[0]);
      });
      return ret;
    },

    copy() {
      navigator.clipboard.writeText(this.code);
    },

    copyResponse: function () {},

    //先頭文字を大文字に変換するfunction
    toUpperFirstLetter: function (str) {
      return str.charAt(0).toUpperCase() + str.substring(1).toLowerCase();
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
