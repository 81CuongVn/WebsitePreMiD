<template>
	<div class="staff-container">
		<div
			class="staffMember"
			v-for="sU in staffMembers"
			:key="sU.user.id"
			:style="{ 'box-shadow': `${sU.user.roleColor} 0px 0px 7px;` }"
		>
			<div
				class="bg"
				:style="{
					'background-image': `url(${sU.user.avatar.replace('gif', 'png')})`
				}"
			></div>
			<div class="userInfo">
				<img :src="sU.user.avatar" />
				<h1>
					{{ sU.user.name }}
					<p :style="{ color: sU.user.roleColor }">{{ sU.user.role }}</p>
				</h1>
			</div>
			<div class="chart-container" v-if="loaded">
				<LineChart
					v-if="sU.ready"
					:chartdata="sU.chartdata"
					:options="options"
					class="userChart"
				/>
			</div>
		</div>
	</div>
</template>

<script>
	import { Line } from "vue-chartjs";

	export default {
		name: "StaffActivity",
		extends: Line,
		data() {
			return {
				staffMembers: [],
				loaded: false,
				tickets: [],
				allDates: [],
				counts: {},
				options: {
					responsive: true,
					maintainAspectRatio: false,
					scales: {
						yAxes: [
							{
								ticks: {
									fontColor: "#eaeaea",
									stepSize: 2,
									min: 0,
									max: 16
								}
							}
						],
						xAxes: [
							{
								ticks: {
									fontColor: "#c3c3c3"
								}
							}
						]
					}
				}
			};
		},
		beforeMount() {
			this.$graphql(
				`{
					tickets(token: "${this.$auth.$storage._state["_token.discord"]}") {
						ticketId
						userId
						timestamp
						created
						accepter
						status
						supporters
						lastUserMessage
						messages {
							userId
							content
							sent
						}
					}
					credits {
						user {
							id
							name
							tag
							avatar
							role
							roleColor
						}
						roles {
							id
						}
					}
				}
				`
			).then(async data => {
				let credits = data.credits;
				let tickets = data.tickets;

				credits.map(user => {
					if (
						user.roles.find(r => r.id === "566417964820070421") !== undefined
					) {
						this.staffMembers.push(user);
					}
				});

				this.tickets = tickets;

				this.staffMembers.map(sU => {
					sU.tickets = this.tickets.filter(
						ticket => ticket.accepter === sU.user.id
					);

					sU.allDates = [];

					sU.tickets.map(ticket => {
						let date = new Date(ticket.timestamp);
						let fDate = this.formatDate(date);

						sU.allDates.push(fDate);
					});

					sU.counts = {};

					sU.allDates.map(d => {
						sU.counts[d] = (sU.counts[d] || 0) + 1;
					});

					let last14Days = this.last14Days(),
						acceptedTickets = [];

					sU.chartdata = {
						labels: [],
						datasets: [
							{
								label: "Accepted tickets",
								backgroundColor: "rgba(114, 137, 218, 0.5)",
								borderColor: "rgba(1, 116, 188, 0.50)",
								pointBackgroundColor: "#b3aeff",
								fontColor: "white",
								data: []
							}
						]
					};

					sU.chartdata.labels = last14Days;

					last14Days.forEach(d => {
						if (sU.counts[d]) acceptedTickets.push(sU.counts[d]);
						else acceptedTickets.push(0);
					});

					sU.chartdata.datasets[0].data = acceptedTickets;

					sU.ready = true;

					console.log(sU);
				});
				this.loaded = true;
			});
		},
		mounted() {
			this.$parent.sortBy = false;
		},
		methods: {
			formatDate(date) {
				var dd = date.getDate();
				var mm = date.getMonth() + 1;
				var yyyy = date.getFullYear();
				if (dd < 10) {
					dd = "0" + dd;
				}
				if (mm < 10) {
					mm = "0" + mm;
				}
				date = dd + "/" + mm + "/" + yyyy;
				return date;
			},
			last14Days() {
				var result = [];
				for (var i = 0; i < 14; i++) {
					var d = new Date();
					d.setDate(d.getDate() - i);
					result.push(this.formatDate(d));
				}

				result.reverse();

				return result;
			}
		}
	};
</script>

<style lang="scss">
	.staff-container {
		display: flex;
		flex-wrap: wrap;

		.staffMember {
			position: relative;
			display: flex;
			flex-direction: column;
			width: 620px;
			height: 400px;
			background: #121212;
			margin-left: 0.5em;
			margin-bottom: 0.5em;
			text-align: center;
			overflow: hidden;
			border-radius: 3px;

			.bg {
				background-position: center;
				position: absolute;
				top: 0;
				left: 0;
				width: 100%;
				height: 100%;
				background-size: contain;
				background-repeat: no-repeat;
				filter: blur(130px);
			}

			.userInfo {
				display: flex;
				justify-content: center;
				padding: 1em;
				z-index: 99;

				h1 {
					text-align: left;
					font-size: 1.25em;
					color: white;
					text-transform: uppercase;

					p {
						text-transform: none;
						font-size: 0.75em;
						margin: 0;
					}
				}

				img {
					border-radius: 50%;
					width: 69px;
					height: 69px;
					margin-right: 0.5em;
				}
			}

			.chart-container {
				display: flex;
				height: 300px;
				width: 100%;
				z-index: 99;

				.userChart {
					height: 300px;
					width: 100%;
				}
			}
		}
	}
</style>
