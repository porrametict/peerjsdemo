<template>
  <div>
    <h1>Server</h1>
    <br />Connect To :
    <input type="text" v-model="peerConnectTo" />
    <button @click="connectPeer">connect</button>

    <hr />

    <button @click="startSteam">start Steam</button>

    <p>Your Video</p>

    <video ref="video" style="max-height: 300px; max-width: 400px;"></video>
  </div>
</template>
<script>
import Peer from "peerjs";
export default {
  name: "Server",
  data: () => ({
    peer: null,

    peerId: "Stream_1",
    peerConnectTo: "Stream_2",
  }),
  created() {
    this.initPeer();
    
  },
  methods: {
    initPeer() {
      this.peer = new Peer(this.peerId);
      this.peer.on("open", function (id) {
        let peerId = id;
        console.log("Ask your friend to join using your peer ID: " + peerId);
      });
      this.peer.on("error", function (err) {
        console.log("error" + err);
      });
    },

    connectPeer() {
      let conn = this.peer.connect(this.peerConnectTo);
      
      // on open will be launch when you successfully connect to PeerServer
      conn.on("open", function () {
        // here you have conn.id
        conn.send("hi! i'm server");
      });

      this.peer.on("connection", function (conn) {
        conn.on("data", function (data) {
          console.log("server receive : ", data);
        });
      });
    },

    async startSteam() {
      let stream = await navigator.mediaDevices.getDisplayMedia({
        video: true,
        // audio: true
      });
      let call = this.peer.call(this.peerConnectTo, stream);
      call.on("stream", (remoteStream) => {
        console.log(remoteStream);
        let video = this.$refs["video"];
        video.srcObject = stream;
        video.play();
      });
    },
  },
};
</script>
