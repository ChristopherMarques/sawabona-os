<script setup lang="ts">
function useGreetings() {
	type Message = string;

	const messages: Message[] = [
		"Lembre-se: <em>todo desafio</em> é mais fácil quando enfrentado em equipe.",
		"Sua visão é nossa missão. Juntos vamos criar <em>algo extraordinário</em>.",
		"Os melhores projetos são <em>movidos por parcerias</em>. Que seja mais um dia de sucesso juntos!",
		"Cada detalhe, cada ideia, cada feedback. <em>Tudo importa</em>. Obrigado por confiar sua visão a nós.",
		"A gestão de projetos vai além do destino: é sobre <em>curtir e aprender com a jornada</em>. É um prazer estar nessa com você.",
		"A cada desafio, <em>nos tornamos mais fortes</em>. Obrigado por nos impulsionar a dar o nosso melhor!",
		"Do conceito à entrega, seus sonhos estão em <em>boas mãos</em>. Vamos continuar avançando juntos.",
		"Como numa sinfonia, <em>cada nota conta</em>. Estamos em sintonia com sua visão, criando uma verdadeira obra-prima.",
		"Enquanto enfrentamos as tarefas de hoje, saiba que nossa <em>dedicação</em> ao seu sucesso é total.",
		"Cada passo com você é um passo rumo à <em>excelência</em>. Obrigado por tornar essa jornada mais brilhante.",
	];

	function getTodaysMessage(): Message {
		const now = new Date();
		const start = new Date(now.getFullYear(), 0, 0);
		const difference = now.getTime() - start.getTime();
		const oneDay = 1000 * 60 * 60 * 24;
		const dayOfYear = Math.floor(difference / oneDay);
		const messageIndex = dayOfYear % messages.length;

		return messages[messageIndex];
	}

	return {
		getTodaysMessage,
	};
}

function greetUser() {
	const hour = new Date().getHours();
	if (hour < 12) return 'Bom dia';
	if (hour < 18) return 'Boa tarde';
	return 'Boa noite';
}

const { getTodaysMessage } = useGreetings();
const { user } = useDirectusAuth();
</script>

<template>
	<PageContainer>
		<img class="w-48 ml-auto mr-0" src="~/assets/illustrations/tokyo-luminous-table-lamp-on-boxes.svg" />
		<TypographyTitle class="normal-case">{{ greetUser() }} {{ user?.first_name ?? 'amigo(a)' }},</TypographyTitle>
		<TypographyHeadline :content="getTodaysMessage()" size="xl" />
		<VDivider class="my-8" />
		<div class="grid w-full grid-cols-1 gap-6 md:grid-cols-2">
			<PortalInvoiceWidget />
			<PortalTaskWidget />
		</div>
	</PageContainer>
</template>
