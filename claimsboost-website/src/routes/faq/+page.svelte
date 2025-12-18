<script>
	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';

	let openItems = $state({});

	const faqs = [
		{
			id: 1,
			question: 'What is ClaimsBoost?',
			answer: 'ClaimsBoost is a free platform that helps people understand their legal options after an injury or accident. We provide settlement estimates based on real case data and help you find top-rated personal injury lawyers in your area.'
		},
		{
			id: 2,
			question: 'How much does it cost to get an estimate?',
			answer: 'Absolutely nothing! Our case estimate tool is completely free to use and comes with no obligations. We believe everyone deserves to know their rights and potential compensation.'
		},
		{
			id: 3,
			question: 'How accurate are the settlement estimates?',
			answer: 'Our estimates are based on thousands of real settlement cases with similar circumstances to yours. While every case is unique, our AI analyzes factors like injury type, severity, location, and liability to provide a realistic range based on actual historical outcomes.'
		},
		{
			id: 4,
			question: 'Is my information private and secure?',
			answer: 'Yes, your privacy is our priority. Your information is encrypted and never shared unless you explicitly choose to connect with a law firm. We do not sell your data to third parties.'
		},
		{
			id: 5,
			question: 'How long does the process take?',
			answer: 'You can get an initial estimate in just 2 minutes. If you choose to connect with a lawyer, most offer same-day consultations. The full legal process varies by case complexity.'
		},
		{
			id: 6,
			question: 'What types of cases do you cover?',
			answer: 'We primarily focus on personal injury cases including car accidents, truck accidents, motorcycle accidents, slip and falls, medical malpractice, workplace injuries, product liability, and more.'
		},
		{
			id: 7,
			question: 'What type of law firms are on ClaimsBoost?',
			answer: 'We feature top-rated personal injury law firms that specialize in cases like yours. All firms are vetted for their track record, client reviews, and professional standing. From boutique practices to established firms, each one has proven experience winning settlements for their clients.'
		},
		{
			id: 8,
			question: 'Do I have to hire a lawyer after getting an estimate?',
			answer: 'No, there is no obligation to hire a lawyer. The estimate is for informational purposes only. You can use it to understand your situation better and decide if pursuing legal action is right for you.'
		},
		{
			id: 9,
			question: 'How do law firms get listed on ClaimsBoost?',
			answer: 'Law firms are listed based on publicly available information, including their practice areas, location, and client reviews. Firms can also claim their profiles to ensure their information is accurate and up-to-date.'
		},
		{
			id: 10,
			question: 'I work at a law firm. How do I claim our profile?',
			answer: 'You can claim your law firm\'s profile by visiting your firm\'s page on ClaimsBoost and clicking the "Claim this profile" link at the bottom of the contact section. We\'ll verify your association with the firm before granting access.'
		}
	];

	// SEO: FAQPage structured data for rich snippets
	const faqSchema = {
		"@context": "https://schema.org",
		"@type": "FAQPage",
		"mainEntity": faqs.map(faq => ({
			"@type": "Question",
			"name": faq.question,
			"acceptedAnswer": {
				"@type": "Answer",
				"text": faq.answer
			}
		}))
	};

	function toggleItem(id) {
		openItems[id] = !openItems[id];
	}
</script>

<svelte:head>
	<title>Frequently Asked Questions - ClaimsBoost</title>
	<meta name="description" content="Find answers to common questions about ClaimsBoost, settlement estimates, finding a lawyer, and how our platform works." />
	{@html `<script type="application/ld+json">${JSON.stringify(faqSchema)}</script>`}
</svelte:head>

<div class="page">
	<Header />

	<main>
		<section class="faq-section">
			<div class="container">
				<h1>Frequently Asked Questions</h1>
				<p class="subtitle">Have questions? We have answers. Find more information below.</p>

				<div class="faq-list">
					{#each faqs as faq}
						<div class="faq-item" class:active={openItems[faq.id]}>
							<button
								class="faq-question"
								onclick={() => toggleItem(faq.id)}
								aria-expanded={openItems[faq.id] || false}
							>
								<span>{faq.question}</span>
								<svg
									class="faq-icon"
									class:rotated={openItems[faq.id]}
									width="24"
									height="24"
									viewBox="0 0 24 24"
									fill="none"
									stroke="currentColor"
									stroke-width="2"
								>
									<path d="M6 9l6 6 6-6"/>
								</svg>
							</button>
							{#if openItems[faq.id]}
								<div class="faq-answer">
									<p>{faq.answer}</p>
								</div>
							{/if}
						</div>
					{/each}
				</div>

				<div class="contact-cta">
					<p>Still have questions?</p>
					<a href="/contact" class="contact-link">Contact us</a>
				</div>
			</div>
		</section>
	</main>

	<Footer />
</div>

<style>
	.page {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		background: white;
	}

	main {
		flex: 1;
	}

	.faq-section {
		padding: 60px 20px 80px;
		background: #f9f9f9;
	}

	.container {
		max-width: 800px;
		margin: 0 auto;
	}

	h1 {
		font-size: 32px;
		font-weight: 700;
		margin-bottom: 12px;
		text-align: center;
		color: #1a1a1a;
	}

	.subtitle {
		text-align: center;
		color: #666;
		font-size: 16px;
		margin-bottom: 40px;
	}

	.faq-list {
		border-top: 1px solid #e5e5e5;
		background: white;
		border-radius: 12px;
		overflow: hidden;
		box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
	}

	.faq-item {
		border-bottom: 1px solid #e5e5e5;
		transition: background 0.2s;
	}

	.faq-item:last-child {
		border-bottom: none;
	}

	.faq-item.active {
		background: #f8f9fa;
	}

	.faq-question {
		width: 100%;
		padding: 20px;
		background: none;
		border: none;
		text-align: left;
		cursor: pointer;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 16px;
		font-weight: 500;
		color: #1a1a1a;
		font-family: inherit;
		transition: color 0.2s;
		gap: 16px;
	}

	.faq-question:hover {
		color: #FF7B00;
	}

	.faq-question span {
		flex: 1;
	}

	.faq-icon {
		flex-shrink: 0;
		transition: transform 0.3s ease;
		color: #666;
	}

	.faq-icon.rotated {
		transform: rotate(180deg);
	}

	.faq-answer {
		padding: 0 20px 20px;
		animation: slideDown 0.3s ease;
	}

	.faq-answer p {
		color: #666;
		line-height: 1.6;
		margin: 0;
	}

	@keyframes slideDown {
		from {
			opacity: 0;
			transform: translateY(-10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}

	/* Contact CTA */
	.contact-cta {
		margin-top: 40px;
		text-align: center;
		display: flex;
		align-items: center;
		justify-content: center;
		gap: 8px;
	}

	.contact-cta p {
		color: #666;
		font-size: 16px;
		margin: 0;
	}

	.contact-link {
		color: #2563eb;
		font-size: 16px;
		font-weight: 500;
		text-decoration: none;
		transition: color 0.2s;
	}

	.contact-link:hover {
		color: #1d4ed8;
		text-decoration: underline;
	}

	/* Responsive */
	@media (min-width: 768px) {
		h1 {
			font-size: 36px;
		}

		.subtitle {
			font-size: 18px;
		}

		.faq-question {
			font-size: 18px;
			padding: 24px;
		}

		.faq-answer {
			padding: 0 24px 24px;
		}
	}

	@media (min-width: 1024px) {
		.faq-section {
			padding: 80px 20px 100px;
		}

		h1 {
			font-size: 44px;
		}
	}
</style>
