<template>
  <div class="container mt-5">
    <h2 class="text-center">SQLITE VIEWER</h2>
    <div class="form-group">
      <label for="file-upload" class="btn btn-primary">
        UPLOAD
      </label>
      <input type="file" id="file-upload" @change="handleFileUpload" class="d-none" />
    </div>
    <div v-if="errorMessage" class="alert alert-danger">{{ errorMessage }}</div>
    <div v-if="tableData.length">
      <h4>Dados da Tabela:</h4>
      <table class="table table-dark">
        <thead>
          <tr>
            <th v-for="(value, key) in tableData[0]" :key="key">{{ key }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, index) in tableData" :key="index">
            <td v-for="(value, key) in row" :key="key">{{ value }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import initSqlJs from 'sql.js';

export default {
  data() {
    return {
      tableData: [],
      errorMessage: ''
    };
  },
  methods: {
    async handleFileUpload(event) {
      const file = event.target.files[0];
      const fileName = file.name.toLowerCase();

      if (fileName.endsWith('.sqlite') || fileName.endsWith('.db')) {
        try {
          const buffer = await file.arrayBuffer();
          const SQL = await initSqlJs({ locateFile: file => `https://sql.js.org/dist/${file}` });
          const db = new SQL.Database(new Uint8Array(buffer));

          this.tableData = this.extractTableData(db);
        } catch (error) {
          this.errorMessage = 'Erro ao processar o arquivo SQLite.';
        }
      } else {
        this.errorMessage = 'Formato de arquivo não suportado. Por favor, envie um arquivo SQLite (.sqlite ou .db).';
      }
    },
    extractTableData(db) {
      try {
        const result = db.exec("SELECT * FROM sqlite_master WHERE type='table';");
        if (result.length === 0) {
          this.errorMessage = 'Nenhuma tabela encontrada no banco de dados SQLite.';
          return [];
        }

        const tableName = result[0].values[0][1]; 
        const tableResult = db.exec(`SELECT * FROM ${tableName};`);

        if (tableResult.length === 0) {
          this.errorMessage = `Nenhum dado encontrado na tabela ${tableName}.`;
          return [];
        }

        const columns = tableResult[0].columns;
        const values = tableResult[0].values;

        return values.map(row => {
          let rowData = {};
          columns.forEach((col, i) => {
            rowData[col] = row[i];
          });
          return rowData;
        });
      } catch (error) {
        this.errorMessage = 'Erro ao extrair dados da tabela.';
        return [];
      }
    }
  }
};
</script>
