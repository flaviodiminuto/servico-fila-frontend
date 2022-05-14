<template>
	<b-container id="conteudo">
		<b-row class="bv-example-row">
			<b-col cols="10">
					<b-form-input id="input-1" type="text" placeholder="Nome do Cliente" v-model="nome_cliente" required>
					</b-form-input>
			</b-col>
			<b-col>
				<a
				href="#"
				@click="submeterPedido()">
					Adicionar
				</a>
			</b-col>
		</b-row>
		<b-row>
			<h2 id="pedidos-titulo">Lista de pedidos:</h2>
		</b-row>
		<b-row id="lista-pedidos">
			<b-col id="em-preparo">
				<h3>Em Preparo
				<b-badge variant="light">{{pedidos_em_preparo.length}} <span class="sr-only">Pedidos em preparo</span></b-badge></h3>
				<b-list-group>
					<b-list-group-item
					href="#"
					v-for = "(pedido) in pedidos_em_preparo"
					:key="pedido.id"
					@click="atualizarStatusPedido(pedido.id)">
						{{pedido.nome_cliente}}
					</b-list-group-item>
				</b-list-group>
			</b-col>
			<b-col id="pronto-para-retirar">
				<h3>Pronto Para Retirar
				<b-badge variant="light">{{pedidos_esperando_retirada.length}} <span class="sr-only">Pronto Para Retirar</span></b-badge></h3>
				<b-list-group>
					<b-list-group-item
					href="#"
					v-for = "(pedido) in pedidos_esperando_retirada"
					:key="pedido.id"
					@click="atualizarStatusPedido(pedido.id)">
						{{pedido.nome_cliente}}
					</b-list-group-item>
				</b-list-group>
			</b-col>
		</b-row>
	</b-container>
</template>

<script>
	import axios from 'axios'
	
	export default {
		name: 'HelloWorld',
		props: {
			msg: String
		},
		data() {
			return {
				service_url: 'http://localhost:8090', /* URL e porta do seu serviço */
				nome_cliente: '',
				fila: Object,
				pedidos_em_preparo: [],
				pedidos_esperando_retirada: []
			}
		},
		mounted() {
			this.getfilas()
		},
		methods: {
			onSubmit(event) {
				event.preventDefault()
				alert(JSON.stringify(this.form))
			},
			getfilas(){
				axios
					.get(this.service_url + '/filas')
					.then(response => {
						this.fila = response.data
						if(this.fila){
							this.delay(500)
							.then(()=>{
								this.getPedidosProntosParaRetirada(this.fila.id)
							})
						}
					})
			},
			getPedidosEmPreparo(fila_id){
				axios
					.get(this.service_url + '/filas/'+fila_id+'/pedidos')
					.then(response => {
						this.pedidos_em_preparo = response.data
					})
			},
			getPedidosProntosParaRetirada(fila_id){
				axios
					.get(this.service_url + '/filas/'+fila_id+'/pedidos', {params: { status: 1}})
					.then(response => {
						this.pedidos_esperando_retirada = response.data
						this.getPedidosEmPreparo(this.fila.id)
					})
			},
			limparCampoNome(){
				this.nome_cliente = ''
			},
			submeterPedido(){
				const pedido = {'nome_cliente': this.nome_cliente}
				axios
					.post(this.service_url + '/pedidos', pedido)
					.then(response => {
						console.log(response)
					})
					.catch(error => {
						console.log(error)
					})
				this.getfilas()
				this.nome_cliente = ''
			},
			atualizarStatusPedido(pedido_id){
				const rota = this.service_url + '/pedidos/' + pedido_id + '/status'
				axios
					.patch(rota)
					.then(response =>{
						if(response.status == 200){
							this.getfilas()
						}
					})
			},
			delay(time){
				return new Promise(resolve => setTimeout(resolve, time));
			}
		}
	}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
div{
	/* border: solid 1px black; */
}
.list-group{
	overflow: auto;
	height: 500px;
}
#em-preparo > div > a:hover,  #pronto-para-retirar > div > a:hover {
	color:#5555FF;
	font-weight: 700;
}
#conteudo{
	padding: 30px;
	border-radius: 15px;
	border: 2px solid #73AD21;
}

#em-preparo, #pronto-para-retirar{
	margin: 5px;
	padding: 10px;
	border-radius: 15px;
	border: 1px solid #73AD21;
	height: 600px;
}

#pedidos-titulo {
	margin-top: 50px;
	margin-bottom: 50px;
}
</style>
