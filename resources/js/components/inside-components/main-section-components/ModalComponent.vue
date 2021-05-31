
<template>
  <transition name="modal-fade">
    <div class="modal-backdrop">
      <div class="modal" role="dialog" aria-labelledby="modalTitle" aria-describedby="modalDescription">
        <header class="modal-header" id="modalTitle">
          <slot name="header">
            Выберите ваш режим игры
            <button type="button" class="btn-close" @click="close" aria-label="Close modal">x</button>
          </slot>
        </header>
        <section class="modal-body" id="modalDescription">
          <form class="checker-block" action="#">
              <input id="radio-4" type="radio" name="radio" v-bind:value='100' v-model="gamePrice" checked>
              <label for="radio-4">100р</label>
                  
              <input id="radio-5" type="radio" name="radio" v-bind:value='500' v-model="gamePrice">
              <label for="radio-5">500р</label>
                    
              <input id="radio-6" type="radio" name="radio" v-bind:value='1000' v-model="gamePrice">
              <label for="radio-6">1000р</label>
          </form>	
          <button class="join" @click="sendData">Вступить в игру</button>
        </section>
      </div>
    </div>
  </transition>
</template>

<script>
  export default {
    props: ['text', 'user'],
    data(){
      return{
        gamePrice: 0,
        id: 0
      }
    },
    methods: {
      close() {
        this.$emit('close');
      },
      sendData() {
                axios.post("http://127.0.0.1:3000/"+this.text+"-queue/", {username: this.user.name, price: this.gamePrice})
                .then((response) => {
                    // подключение к комнате очередей
                    socket.emit('joinQueue', {queue_id: response.data.queue_id});
                    if(response.data.chat_id != 0){
                        // запрос в случае создания чата
                        socket.emit('queueMessage', {queue_id: response.data.queue_id, chat_id: response.data.chat_id})
                    }
                    socket.on('queue_message', message => {
                        // запрос для записи параметров БД
                        axios({
                            method: 'post',
                            url: 'game/id-defender',
                            data: { chat_id: message}
                        })
                        .then((response)=>{
                          console.log("работает")
                          window.location.replace('http://127.0.0.1:8000/chat/'+message);
                        });
                    })
                })
            }
    }
  };
</script>

<style scoped>
  .modal-fade-enter,
  .modal-fade-leave-active {
    opacity: 0;
  }

  .modal-fade-enter-active,
  .modal-fade-leave-active {
    transition: opacity .5s ease
  }

  .modal-backdrop {
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
  }

  .modal {
    position: absolute;
    top: 35%;
    left: 35%;
    background: #FFFFFF;
    box-shadow: 20px 20px 200px 5px;
    overflow-x: auto;
    display: flex;
    flex-direction: column;
    width: 30%;
    height: min-content;
    background: #272727;
    border-radius: 15px;
  }

  .modal-header,
  .modal-footer {
    padding: 15px;
    display: flex;
  }

  .modal-header {
    border-bottom: 1px solid #eeeeee;
    color: #fff;
    justify-content: space-between;
  }

  .modal-header button{
    padding: 0px;
    line-height:20px;
    color: #C04149;
  }

  .modal-footer {
    border-top: 1px solid #eeeeee;
    justify-content: flex-end;
  }

  .modal-body {
    position: relative;
    padding: 20px 10px;
    display: flex;
    flex-direction: column;
    align-items: center;
    color: #fff
  }

  .btn-close {
    border: none;
    font-size: 20px;
    padding: 20px;
    cursor: pointer;
    font-weight: bold;
    color: #4AAE9B;
    background: transparent;
  }

  .btn-green {
    color: white;
    background: #4AAE9B;
    border: 1px solid #4AAE9B;
    border-radius: 2px;
  }
  .checker-block{
	display: flex ;
	justify-content: space-around;
	width: 100%;
	flex-wrap: wrap;
	margin-top: 15px;
}

.checker-block input[type=radio] {
	display: none;
}
.checker-block label {
	display: inline-block;
	cursor: pointer;
	padding: 0px 15px;
	line-height: 30px;
	border-radius: 50px;
	user-select: none;
	background: #6A8ED4;
	transition: 0.3s;
	color: #fff;
	width: 140px;
  height: 30px;
	text-align: center;
}

.checker-block input[type=radio]:checked + label {
	background: #C04149;
}

.join{
	color: #fff;
	background: #C04149;
	transition: 0.3s;
	width: 140px;
	text-align: center;
	border: none;
	border-radius: 50px;
	height: 30px;
	margin-top: 20px;
}

</style>