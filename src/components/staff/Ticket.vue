<template>
	<div class="staff-container">
		<div class="messages" v-if="ready">
			<p class="action">wip</p>
			<p
				class="action"
				v-if="supporters.find(s => s.userId == ticket.accepter)"
			>
				Ticket accepted by:
				{{ supporters.find(s => s.userId == ticket.accepter).username }}
			</p>
			<div
				v-for="message in ticket.messages"
				:key="message.content"
				:class="{
					right:
						message.userId == ticket.accepter ||
						supporters.find(s => s.userId == message.userId)
				}"
			>
				<Message
					v-if="message.userId == ticket.accepter"
					:message="message"
					:user="{ avatar: ticket.supporterAvatar, name: ticket.supporterName }"
					:badge="{ type: 'accepter', color: '#6b0909' }"
				/>

				<Message
					v-else-if="message.userId == ticket.userId"
					:message="message"
					:user="{
						avatar: ticket.userAvatar || 'https://i.imgur.com/zsd0gU4.png',
						name: ticket.userName || 'User'
					}"
				/>

				<Message
					v-else-if="
						ticket.supporters &&
						ticket.supporters.includes(message.userId) &&
						ticket.accepter !== message.userId
					"
					:message="message"
					:set="
						(s = ticket.supportersInfo.find(s => s.user.id == message.userId)
							.user)
					"
					:user="{
						avatar: s.avatar,
						name: s.name
					}"
					:badge="{ type: 'supporter', color: '#48d41e' }"
				/>
			</div>
			<p
				class="action"
				v-if="ticket.closer && supporters.find(s => s.userId == ticket.closer)"
			>
				Ticket closed by:
				{{ supporters.find(s => s.userId == ticket.closer).username }}
			</p>
		</div>
	</div>
</template>

<script>
	import moment from "moment";

	export default {
		name: "Ticket",
		props: {
			ticket: Object
		},
		data() {
			return {
				discordUsers: [],
				supporters: [],
				ready: false,
				moment: moment
			};
		},
		mounted() {
			this.$axios(`/v2/discordUsers`).then(response => {
				this.discordUsers = response.data;

				if (this.ticket.supporters)
					this.ticket.supporters.forEach(s => {
						let supporter = this.discordUsers.find(u => u.userId == s);
						this.supporters.push(supporter);
					});

				this.ready = true;
			});
		}
	};
</script>

<style lang="scss" scoped>
	.staff-container {
		justify-content: normal;

		.action {
			text-align: center;
		}

		.messages {
			display: flex;
			flex-direction: column;
			width: 100%;

			.message {
				padding: 0.75em;
				background: #313131;
				width: 600px;
				float: left;
				margin-bottom: 1em;
				border-radius: 3px;
				color: white;

				.userAvatar {
					height: 100%;
					img {
						width: 50px;
						height: 50px;
						float: left;
						border-radius: 50%;
						margin-right: 1em;
					}

					.role {
						color: black;
						font-size: 0.7em;
						margin-left: 0.5em;
						border-radius: 2px;
						padding: 0.2em;
					}

					.supporter {
						background: #48d41e;
					}

					.accepter {
						background: #6b0909;
					}
				}

				.time {
					color: #c3c3c3;
					font-size: 0.7em;
				}
			}

			.right {
				margin-left: auto;
			}

			.input {
				position: fixed;
				bottom: 90px;
				width: 100%;
				height: 75px;
				background: #121212;
			}
		}
	}
</style>
