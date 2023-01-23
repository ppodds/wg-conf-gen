<template>
  <v-app>
    <!-- Sizes your content based upon application components -->
    <v-main>
      <!-- Provides the application the proper gutter -->
      <v-container fluid>
        <v-tabs v-model="tab">
          <v-tab value="Server">Server</v-tab>
          <v-tab v-for="(peer, i) of peersConfig" :key="i" :value="`Peer${i}`">
            Peer{{ i + 1 }}
          </v-tab>
        </v-tabs>
        <v-window v-model="tab">
          <v-window-item value="Server">
            <v-row>
              <v-col>
                <v-form>
                  <v-text-field
                    label="Endpoint IP"
                    v-model="serverConfig.endPointIP"
                  >
                  </v-text-field>
                  <v-text-field
                    label="Address"
                    v-model="serverConfig.address"
                  ></v-text-field>
                  <v-text-field
                    label="Public Key"
                    v-model="serverConfig.publicKey"
                  ></v-text-field>
                  <v-text-field
                    label="Private Key"
                    v-model="serverConfig.privateKey"
                  ></v-text-field>
                  <v-text-field
                    label="Preshared Key"
                    v-model="serverConfig.presharedKey"
                  ></v-text-field>
                  <v-text-field
                    label="Listen Port"
                    type="number"
                    v-model="serverConfig.listenPort"
                  ></v-text-field>
                  <v-text-field
                    label="MTU"
                    type="number"
                    v-model="serverConfig.mtu"
                  ></v-text-field>
                  <multiple-input
                    label="PreUp Script"
                    v-model:items="serverConfig.preUpScripts"
                  />
                  <multiple-input
                    label="PostUp Script"
                    v-model:items="serverConfig.postUpScripts"
                  />
                  <multiple-input
                    label="PreDown Script"
                    v-model:items="serverConfig.preDownScripts"
                  />
                  <multiple-input
                    label="PostDown Script"
                    v-model:items="serverConfig.postDownScripts"
                  />
                  <v-row>
                    <v-btn @click="serverConfig.preUpScripts.push('')">
                      Add PreUp Script
                    </v-btn>
                  </v-row>
                  <v-row>
                    <v-btn @click="serverConfig.postUpScripts.push('')">
                      Add PostUp Script
                    </v-btn>
                  </v-row>
                  <v-row>
                    <v-btn @click="serverConfig.preDownScripts.push('')">
                      Add PreDown Script
                    </v-btn>
                  </v-row>
                  <v-row>
                    <v-btn @click="serverConfig.postDownScripts.push('')">
                      Add PostDown Script
                    </v-btn>
                  </v-row>
                  <v-switch
                    v-model="serverConfig.saveConfig"
                    label="Save Config"
                  ></v-switch>
                  <v-btn @click="addPear"> Add Peer </v-btn>
                </v-form>
              </v-col>
              <v-divider vertical></v-divider>
              <v-col>
                <h2>
                  Server Config
                  <v-icon
                    icon="mdi-content-copy"
                    size="x-small"
                    @click="copyToClipboard(generatedServerConfig)"
                  ></v-icon>
                </h2>
                <code>
                  <pre>{{ generatedServerConfig }}</pre>
                </code>
              </v-col>
            </v-row>
          </v-window-item>
          <v-window-item
            v-for="(peer, i) of peersConfig"
            :key="i"
            :value="`Peer${i}`"
          >
            <v-row>
              <v-col>
                <v-form>
                  <v-text-field
                    label="Public Key"
                    v-model="peer.publicKey"
                  ></v-text-field>
                  <v-text-field
                    label="Private Key"
                    v-model="peer.privateKey"
                  ></v-text-field>
                  <v-text-field
                    label="Address"
                    v-model="peer.address"
                  ></v-text-field>
                  <v-text-field label="DNS" v-model="peer.dns"></v-text-field>
                  <v-text-field
                    label="Allowed IPs"
                    v-model="peer.allowedIps"
                  ></v-text-field>
                  <v-btn @click="removePeer(i)"> Remove </v-btn>
                </v-form>
              </v-col>
              <v-divider vertical></v-divider>
              <v-col>
                <h2>
                  Peer {{ i + 1 }} Config
                  <v-icon
                    icon="mdi-content-copy"
                    size="x-small"
                    @click="copyToClipboard(generatedPeerConfigs[i])"
                  ></v-icon>
                </h2>
                <code>
                  <pre>{{ generatedPeerConfigs[i] }}</pre>
                </code>
              </v-col>
            </v-row>
          </v-window-item>
        </v-window>
      </v-container>
      <v-alert type="success" v-show="alertCounter !== 0">
        Copy Success
      </v-alert>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
const tab = ref(null);
const serverConfig = ref({
  endPointIP: "test.com",
  address: "10.0.0.1",
  listenPort: 51820,
  publicKey: "",
  privateKey: "",
  mtu: 1500,
  presharedKey: "",
  preUpScripts: [] as string[],
  postUpScripts: [
    "iptables -A FORWARD -i %i -j ACCEPT",
    "iptables -A FORWARD -o %i -j ACCEPT",
    "iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE",
  ],
  preDownScripts: [] as string[],
  postDownScripts: [
    "iptables -D FORWARD -i %i -j ACCEPT",
    "iptables -D FORWARD -o %i -j ACCEPT",
    "iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE",
  ],
  saveConfig: false,
});
const peersConfig = ref([
  {
    publicKey: "",
    privateKey: "",
    address: "10.0.0.2",
    allowedIps: "0.0.0.0/0",
    dns: "8.8.8.8",
  },
]);
const generatedServerConfig = computed(() => {
  let result = "[Interface]\n";
  result += `Address = ${serverConfig.value.address}\n`;
  result += `ListenPort = ${serverConfig.value.listenPort}\n`;
  result += `PrivateKey = ${serverConfig.value.privateKey}\n`;
  result += `MTU = ${serverConfig.value.mtu}\n`;
  for (const script of serverConfig.value.preUpScripts)
    result += `PreUp = ${script}\n`;
  for (const script of serverConfig.value.postUpScripts)
    result += `PostUp = ${script}\n`;
  for (const script of serverConfig.value.preDownScripts)
    result += `PreDown = ${script}\n`;
  for (const script of serverConfig.value.postDownScripts)
    result += `PostDown = ${script}\n`;
  result += `SaveConfig = ${serverConfig.value.saveConfig}\n`;

  for (const peer of peersConfig.value) {
    result += "\n[Peer]\n";
    result += `PublicKey = ${peer.publicKey}\n`;
    result += `AllowedIPs = ${peer.address}\n`;
  }
  return result;
});

const generatedPeerConfigs = computed(() => {
  const result = [] as string[];
  for (const peer of peersConfig.value) {
    let config = "[Interface]\n";

    config += `Address = ${peer.address}\n`;
    config += `DNS = ${peer.dns}\n`;
    config += `PrivateKey = ${peer.privateKey}\n`;
    config += "\n[Peer]\n";
    config += `PublicKey = ${serverConfig.value.publicKey}\n`;
    config += `PresharedKey = ${serverConfig.value.presharedKey}\n`;
    config += `AllowedIPs = ${peer.allowedIps}\n`;
    config += `Endpoint = ${serverConfig.value.endPointIP}:${serverConfig.value.listenPort}\n`;
    result.push(config);
  }
  return result;
});

const alertCounter = ref(0);

function addPear() {
  peersConfig.value.push({
    publicKey: "",
    privateKey: "",
    address: "",
    allowedIps: "0.0.0.0/0",
    dns: "8.8.8.8",
  });
}

function removePeer(index: number) {
  peersConfig.value.splice(index, 1);
}

async function copyToClipboard(text: string) {
  await navigator.clipboard.writeText(text);
  alertCounter.value++;
  setTimeout(() => {
    alertCounter.value--;
  }, 3000);
}
</script>

<style scoped>
code > pre {
  background-color: #f3f4f6;
}

.v-alert {
  position: fixed;
  bottom: 10%;
  left: 50%;
  transform: translateX(-50%);
  z-index: 100;
}
</style>
