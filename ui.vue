<template>
  <v-container fluid class="plugin-main">
    <v-row>
      <v-col
        v-for="server in servers"
        :key="server.port"
        cols="3"
      >
        <v-card>
          <v-card-text>
            <div>{{ server.path }}</div>
            <div>:{{ server.port }}</div>
          </v-card-text>

          <v-card-actions>
            <v-spacer />
            <v-btn @click="open(server.port)">查看</v-btn>
            <v-btn @click="closeServer(server.port)">关闭</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
      <v-col cols="3">
        <v-card link class="server-card d-flex align-center justify-center" @click="newServer">
          <v-icon>add</v-icon>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
const vuetify = window.vuetify$.components;

export default {
  name: 'ui',

  components: {
    ...vuetify,
  },

  data: () => ({
    servers: [],
  }),

  methods: {
    async newServer() {
      const result = await window.electron.dialog.showOpenDialog({
        properties: ['openDirectory'],
      });
      if (!result.data.canceled) {
        const port = await window.electron.ipcRenderer.invoke('ipc-fn', {
          type: 'new-server@translime-plugin-static-server',
          args: [result.data.filePaths[0]],
        });
        this.servers.push({
          port: +port,
          path: result.data.filePaths[0],
        });
      }
    },
    async closeServer(port) {
      await window.electron.ipcRenderer.invoke('ipc-fn', {
        type: 'close-server@translime-plugin-static-server',
        args: [port],
      });
    },
    onServerClose() {
      window.electron.ipcRenderer.receive('ipc-reply', ({ type, data }) => {
        if (type === 'server-closed@translime-plugin-static-server') {
          this.servers = this.servers.filter(server => +server.port !== +data.port);
        }
      });
    },
    open(port) {
      window.electron.ipcRenderer.send('ipc-msg', {
        type: 'open-link',
        data: { url: `http://localhost:${port}` },
      });
    },
    async getServers() {
      this.servers = await window.electron.ipcRenderer.invoke('ipc-fn', {
        type: 'get-server-list@translime-plugin-static-server',
        args: [],
      });
    },
  },

  mounted() {
    this.getServers();
    this.onServerClose();
  },
};
</script>

<style scoped>
.plugin-main {
}

.server-card {
  height: 100%;
  min-height: 120px;
}
</style>
