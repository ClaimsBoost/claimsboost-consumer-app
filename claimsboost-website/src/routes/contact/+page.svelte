<script>
	import Header from '$lib/components/Header.svelte';
	import Footer from '$lib/components/Footer.svelte';

	let name = $state('');
	let email = $state('');
	let message = $state('');
	let isSubmitting = $state(false);
	let submitStatus = $state(null); // 'success' | 'error' | null

	async function handleSubmit(event) {
		event.preventDefault();
		isSubmitting = true;
		submitStatus = null;

		try {
			// TODO: Implement actual form submission
			// For now, simulate a delay
			await new Promise(resolve => setTimeout(resolve, 1000));

			// Reset form on success
			name = '';
			email = '';
			message = '';
			submitStatus = 'success';
		} catch (error) {
			console.error('Form submission error:', error);
			submitStatus = 'error';
		} finally {
			isSubmitting = false;
		}
	}
</script>

<svelte:head>
	<title>Contact Us - ClaimsBoost</title>
	<meta name="description" content="Get in touch with ClaimsBoost. We're here to help with questions about finding a personal injury lawyer or using our platform." />
</svelte:head>

<div class="page">
	<Header />

	<main>
		<section class="contact-section">
			<div class="container">
				<h1>Contact Us</h1>
				<p class="subtitle">Have a question or feedback? We'd love to hear from you.</p>

				{#if submitStatus === 'success'}
					<div class="success-message">
						<svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
							<path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/>
							<polyline points="22 4 12 14.01 9 11.01"/>
						</svg>
						<div>
							<strong>Message sent!</strong>
							<p>Thank you for reaching out. We'll get back to you soon.</p>
						</div>
					</div>
				{:else}
					<form class="contact-form" onsubmit={handleSubmit}>
						<div class="form-group">
							<label for="name">Name</label>
							<input
								type="text"
								id="name"
								bind:value={name}
								placeholder="Your name"
								required
								disabled={isSubmitting}
							/>
						</div>

						<div class="form-group">
							<label for="email">Email</label>
							<input
								type="email"
								id="email"
								bind:value={email}
								placeholder="you@example.com"
								required
								disabled={isSubmitting}
							/>
						</div>

						<div class="form-group">
							<label for="message">Message</label>
							<textarea
								id="message"
								bind:value={message}
								placeholder="How can we help you?"
								rows="5"
								required
								disabled={isSubmitting}
							></textarea>
						</div>

						{#if submitStatus === 'error'}
							<div class="error-message">
								Something went wrong. Please try again or email us directly.
							</div>
						{/if}

						<button type="submit" class="submit-button" disabled={isSubmitting}>
							{#if isSubmitting}
								<span class="spinner"></span>
								Sending...
							{:else}
								Send Message
								<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
									<path d="M5 12h14M12 5l7 7-7 7"/>
								</svg>
							{/if}
						</button>
					</form>
				{/if}
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

	.container {
		max-width: 600px;
		margin: 0 auto;
		padding: 0 20px;
	}

	/* Contact Section */
	.contact-section {
		padding: 60px 20px 80px;
		text-align: center;
	}

	.contact-section h1 {
		font-size: 36px;
		font-weight: 700;
		margin-bottom: 16px;
		color: #1a1a1a;
	}

	.subtitle {
		font-size: 18px;
		color: #666;
		margin-bottom: 40px;
	}

	/* Form Styles */
	.contact-form {
		text-align: left;
	}

	.form-group {
		margin-bottom: 24px;
	}

	.form-group label {
		display: block;
		font-size: 14px;
		font-weight: 600;
		color: #1a1a1a;
		margin-bottom: 8px;
	}

	.form-group input,
	.form-group textarea {
		width: 100%;
		padding: 14px 16px;
		font-size: 16px;
		font-family: inherit;
		border: 1px solid #e5e7eb;
		border-radius: 8px;
		background: #fff;
		color: #1a1a1a;
		transition: border-color 0.2s, box-shadow 0.2s;
	}

	.form-group input:focus,
	.form-group textarea:focus {
		outline: none;
		border-color: #2563eb;
		box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
	}

	.form-group input::placeholder,
	.form-group textarea::placeholder {
		color: #9ca3af;
	}

	.form-group input:disabled,
	.form-group textarea:disabled {
		background: #f9fafb;
		cursor: not-allowed;
	}

	.form-group textarea {
		resize: vertical;
		min-height: 120px;
	}

	/* Submit Button */
	.submit-button {
		width: 100%;
		display: inline-flex;
		align-items: center;
		justify-content: center;
		gap: 8px;
		padding: 16px 32px;
		background: linear-gradient(135deg, #FF6800 0%, #FFA500 100%);
		color: white;
		border: none;
		border-radius: 8px;
		font-size: 16px;
		font-weight: 600;
		font-family: inherit;
		cursor: pointer;
		transition: all 0.3s ease;
		box-shadow: 0 4px 15px rgba(255, 104, 0, 0.4);
	}

	.submit-button:hover:not(:disabled) {
		background: linear-gradient(135deg, #FF8000 0%, #FFB733 100%);
		box-shadow: 0 6px 25px rgba(255, 104, 0, 0.5);
		transform: translateY(-2px);
	}

	.submit-button:disabled {
		opacity: 0.7;
		cursor: not-allowed;
		transform: none;
	}

	.submit-button svg {
		transition: transform 0.2s;
	}

	.submit-button:hover:not(:disabled) svg {
		transform: translateX(3px);
	}

	/* Spinner */
	.spinner {
		width: 20px;
		height: 20px;
		border: 2px solid rgba(255, 255, 255, 0.3);
		border-top-color: white;
		border-radius: 50%;
		animation: spin 0.8s linear infinite;
	}

	@keyframes spin {
		to {
			transform: rotate(360deg);
		}
	}

	/* Success Message */
	.success-message {
		display: flex;
		align-items: flex-start;
		gap: 16px;
		padding: 24px;
		background: #ecfdf5;
		border: 1px solid #a7f3d0;
		border-radius: 12px;
		text-align: left;
	}

	.success-message svg {
		flex-shrink: 0;
		color: #10b981;
		margin-top: 2px;
	}

	.success-message strong {
		display: block;
		color: #065f46;
		font-size: 16px;
		margin-bottom: 4px;
	}

	.success-message p {
		color: #047857;
		font-size: 14px;
		margin: 0;
	}

	/* Error Message */
	.error-message {
		padding: 12px 16px;
		background: #fef2f2;
		border: 1px solid #fecaca;
		border-radius: 8px;
		color: #dc2626;
		font-size: 14px;
		margin-bottom: 20px;
	}

	/* Responsive */
	@media (min-width: 768px) {
		.contact-section {
			padding: 80px 20px 100px;
		}

		.contact-section h1 {
			font-size: 44px;
		}

		.subtitle {
			font-size: 20px;
		}
	}
</style>
