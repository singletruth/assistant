<script>
	import { toast } from 'svelte-sonner';

	import { onMount, getContext, tick } from 'svelte';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';

	import { getBackendConfig } from '$lib/apis';
	import { ldapUserSignIn, getSessionUser, userSignIn, userSignUp } from '$lib/apis/auths';

	import { WEBUI_API_BASE_URL, WEBUI_BASE_URL } from '$lib/constants';
	import { WEBUI_NAME, config, user, socket } from '$lib/stores';

	import { generateInitialsImage, canvasPixelTest } from '$lib/utils';

	import Spinner from '$lib/components/common/Spinner.svelte';
	import OnBoarding from '$lib/components/OnBoarding.svelte';

	const i18n = getContext('i18n');

	let loaded = false;

	let mode = $config?.features.enable_ldap ? 'ldap' : 'signin';

	let name = '';
	let email = '';
	let password = '';

	let ldapUsername = '';

	// Image handling
	const pathOptions = [
		'/auth_photo.jpg',                   // Direct root
		'/static/auth_photo.jpg',            // Static directory
		'/static/static/auth_photo.jpg',     // Nested static directory
		'https://images.unsplash.com/photo-1624043305369-e47e15dc5b4f?q=80&w=1976&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D' // Fallback URL
	];
	
	let currentImageIndex = 0;
	let currentImage = pathOptions[currentImageIndex];
	
	function handleImageError() {
		// Try the next image path
		currentImageIndex++;
		if (currentImageIndex < pathOptions.length) {
			currentImage = pathOptions[currentImageIndex];
			console.log('Trying next image path:', currentImage);
		} else {
			console.log('All image paths failed, using final fallback');
			// Use the final fallback
			currentImage = pathOptions[pathOptions.length - 1];
		}
	}
	
	// Background style based on the current image
	$: bgStyle = `background-image: url('${currentImage}'); background-size: cover; background-position: center;`;

	const querystringValue = (key) => {
		const querystring = window.location.search;
		const urlParams = new URLSearchParams(querystring);
		return urlParams.get(key);
	};

	const setSessionUser = async (sessionUser) => {
		if (sessionUser) {
			console.log(sessionUser);
			toast.success($i18n.t(`You're now logged in.`));
			if (sessionUser.token) {
				localStorage.token = sessionUser.token;
			}

			$socket.emit('user-join', { auth: { token: sessionUser.token } });
			await user.set(sessionUser);
			await config.set(await getBackendConfig());

			const redirectPath = querystringValue('redirect') || '/';
			goto(redirectPath);
		}
	};

	const signInHandler = async () => {
		const sessionUser = await userSignIn(email, password).catch((error) => {
			toast.error(`${error}`);
			return null;
		});

		await setSessionUser(sessionUser);
	};

	const signUpHandler = async () => {
		const sessionUser = await userSignUp(name, email, password, generateInitialsImage(name)).catch(
			(error) => {
				toast.error(`${error}`);
				return null;
			}
		);

		await setSessionUser(sessionUser);
	};

	const ldapSignInHandler = async () => {
		const sessionUser = await ldapUserSignIn(ldapUsername, password).catch((error) => {
			toast.error(`${error}`);
			return null;
		});
		await setSessionUser(sessionUser);
	};

	const submitHandler = async () => {
		if (mode === 'ldap') {
			await ldapSignInHandler();
		} else if (mode === 'signin') {
			await signInHandler();
		} else {
			await signUpHandler();
		}
	};

	const checkOauthCallback = async () => {
		if (!$page.url.hash) {
			return;
		}
		const hash = $page.url.hash.substring(1);
		if (!hash) {
			return;
		}
		const params = new URLSearchParams(hash);
		const token = params.get('token');
		if (!token) {
			return;
		}
		const sessionUser = await getSessionUser(token).catch((error) => {
			toast.error(`${error}`);
			return null;
		});
		if (!sessionUser) {
			return;
		}
		localStorage.token = token;
		await setSessionUser(sessionUser);
	};

	let onboarding = false;

	async function setLogoImage() {
		await tick();
		const logo = document.getElementById('logo');

		if (logo) {
			const isDarkMode = document.documentElement.classList.contains('dark');

			if (isDarkMode) {
				const darkImage = new Image();
				darkImage.src = '/static/favicon-dark.png';

				darkImage.onload = () => {
					logo.src = '/static/favicon-dark.png';
					logo.style.filter = ''; // Ensure no inversion is applied if favicon-dark.png exists
				};

				darkImage.onerror = () => {
					logo.style.filter = 'invert(1)'; // Invert image if favicon-dark.png is missing
				};
			}
		}
	}

	onMount(async () => {
		if ($user !== undefined) {
			await goto('/');
		}
		await checkOauthCallback();

		loaded = true;
		setLogoImage();
		
		// Log image path for debugging
		console.log("Initial image path:", currentImage);

		if (($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false) {
			await signInHandler();
		} else {
			onboarding = $config?.onboarding ?? false;
		}
	});
</script>

<svelte:head>
	<title>
		{`${$WEBUI_NAME}`}
	</title>
</svelte:head>

<OnBoarding
	bind:show={onboarding}
	getStartedHandler={() => {
		onboarding = false;
		mode = $config?.features.enable_ldap ? 'ldap' : 'signup';
	}}
/>

<div class="w-full h-screen max-h-[100dvh] relative bg-white dark:bg-gray-950 flex items-center justify-center">
	<div class="w-full absolute top-0 left-0 right-0 h-8 drag-region" />

	{#if loaded}
		<!-- Main Container - No rounded corners on the entire layout -->
		<div class="flex flex-col lg:flex-row w-[95%] sm:w-[90%] max-w-6xl h-[95%] sm:h-[90%] max-h-[800px]">
			<!-- Logo in top left corner - keep as is -->
			<div class="absolute top-4 left-4 sm:top-8 sm:left-10 z-10">
				<img
					id="logo"
					crossorigin="anonymous"
					src="{WEBUI_BASE_URL}/static/splash.png"
					class="w-6 h-6 sm:w-8 sm:h-8"
					alt="logo"
				/>
			</div>
			
			<!-- Left Column - Auth Form -->
			<div class="w-full lg:w-1/2 p-4 sm:p-10 flex flex-col justify-center order-2 lg:order-1">
				<div class="max-w-md mx-auto mt-6">
					<!-- Form content -->
					{#if ($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false}
						<div class="flex items-center gap-3 text-xl sm:text-2xl font-semibold text-gray-800 dark:text-gray-200 mb-6">
							<div>
								{$i18n.t('Signing in to {{WEBUI_NAME}}', { WEBUI_NAME: $WEBUI_NAME })}
							</div>
							<div>
								<Spinner />
							</div>
						</div>
					{:else}
						<div class="mb-6 sm:mb-10">
							<h1 class="text-2xl sm:text-3xl font-bold text-gray-900 dark:text-white mb-2">
								Welcome Back to Singletruth Assistant 👋
							</h1>
							<p class="text-sm sm:text-base text-gray-600 dark:text-gray-400">
								Your trusted data intelligence partner. Sign in to access your customized knowledge systems and harness the power of your organization's information
							</p>
						</div>

						<form 
							class="flex flex-col"
							on:submit={(e) => {
								e.preventDefault();
								submitHandler();
							}}
						>
							{#if $config?.features.enable_login_form || $config?.features.enable_ldap}
								<div class="flex flex-col gap-4 sm:gap-5 mb-6">
									{#if mode === 'signup'}
										<div>
											<label for="name" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
												{$i18n.t('Name')}
											</label>
											<input
												id="name"
												bind:value={name}
												type="text"
												class="w-full px-4 py-2.5 rounded-md border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500"
												autocomplete="name"
												placeholder={$i18n.t('Enter Your Full Name')}
												required
											/>
										</div>
									{/if}

									{#if mode === 'ldap'}
										<div>
											<label for="username" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
												{$i18n.t('Username')}
											</label>
											<input
												id="username"
												bind:value={ldapUsername}
												type="text"
												class="w-full px-4 py-2.5 rounded-md border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500"
												autocomplete="username"
												name="username"
												placeholder={$i18n.t('Enter Your Username')}
												required
											/>
										</div>
									{:else}
										<div>
											<label for="email" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
												{$i18n.t('Email')}
											</label>
											<input
												id="email"
												bind:value={email}
												type="email"
												class="w-full px-4 py-2.5 rounded-md border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500"
												autocomplete="email"
												name="email"
												placeholder="Example@email.com"
												required
											/>
										</div>
									{/if}

									<div>
										<label for="password" class="block text-sm font-medium text-gray-700 dark:text-gray-300 mb-2">
											{$i18n.t('Password')}
										</label>
										<input
											id="password"
											bind:value={password}
											type="password"
											class="w-full px-4 py-2.5 rounded-md border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 text-gray-900 dark:text-white focus:outline-none focus:ring-2 focus:ring-teal-500"
											placeholder="At least 8 characters"
											autocomplete="current-password"
											name="current-password"
											required
										/>
									</div>
								</div>
							{/if}

							<div class="mb-6">
								{#if $config?.features.enable_login_form || $config?.features.enable_ldap}
									{#if mode === 'ldap'}
										<button
											class="w-full px-4 py-3 bg-teal-600 hover:bg-teal-700 text-white font-medium rounded-md transition-colors"
											type="submit"
										>
											{$i18n.t('Authenticate')}
										</button>
									{:else}
										<button
											class="w-full px-4 py-3 bg-teal-600 hover:bg-teal-700 text-white font-medium rounded-md transition-colors"
											type="submit"
										>
											{mode === 'signin'
												? $i18n.t('Sign in')
												: ($config?.onboarding ?? false)
													? $i18n.t('Create Admin Account')
													: $i18n.t('Create Account')}
										</button>

										{#if $config?.features.enable_signup && !($config?.onboarding ?? false)}
											<div class="mt-4 text-sm text-center text-gray-600 dark:text-gray-400">
												{mode === 'signin'
													? $i18n.t("Don't have an account?")
													: $i18n.t('Already have an account?')}

												<button
													class="font-medium text-gray-900 dark:text-gray-300 hover:underline ml-1"
													type="button"
													on:click={() => {
														if (mode === 'signin') {
															mode = 'signup';
														} else {
															mode = 'signin';
														}
													}}
												>
													{mode === 'signin' ? $i18n.t('Sign up') : $i18n.t('Sign in')}
												</button>
											</div>
										{/if}
									{/if}
								{/if}
							</div>
						</form>
						
						<!-- Footer -->
						<div class="mt-auto pt-4 sm:pt-8 text-xs text-center text-gray-500 dark:text-gray-500">
							Singletruth.ai © 2023 ALL RIGHTS RESERVED
						</div>
					{/if}
				</div>
			</div>
			
			<!-- Right Column - Abstract Artwork with rounded corners -->
			<!-- On mobile, show as a small banner at the top -->
			<div class="lg:hidden w-full h-48 overflow-hidden rounded-xl mb-4 order-1">
				<div class="h-full w-full">
					<img 
						src={currentImage} 
						alt="Abstract 3D Artwork" 
						class="object-cover h-full w-full"
						on:error={handleImageError}
					/>
				</div>
			</div>
			
			<!-- Desktop version of image -->
			<div class="hidden lg:block lg:w-1/2 overflow-hidden rounded-xl order-2">
				<div class="h-full w-full">
					<img 
						src={currentImage} 
						alt="Abstract 3D Artwork" 
						class="object-cover h-full w-full"
						on:error={handleImageError}
					/>
				</div>
			</div>
		</div>
	{/if}
</div>
