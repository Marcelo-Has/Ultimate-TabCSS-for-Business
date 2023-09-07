<script>
	import { onMount } from 'svelte';
	import css from '../style.css?inline';
	import copy from 'copy-to-clipboard';
	import Fa from 'svelte-fa';
	import { faCopy, faDownload, faBookmark, faGear } from '@fortawesome/free-solid-svg-icons';
	import Tab from '../components/Tab.svelte';
	import Button from '../components/Button.svelte';
	import Soon from '../../lib/images/under_construction.svg';
	

	let navList = [
		{ name: 'Bookmarks', icon: faBookmark },
		{ name: 'Configuration', icon: faGear }
	];
	let activeItem = navList[0];

	let classInfoList = extractClassInfo(modifyCSS(css));
	let tooltipText = 'Click to copy!';
	let template = 'https://public.tableau.com/workbooks/SuperstoreDashboard_16384653251620.twb';

	onMount(async () => {
		addCss2Html();
	});

	function addCss2Html() {
		// Get the head element from the document
		const head = document.getElementsByTagName('head')[0];
		// Check if a style element with ID 'svelte-style' exists
		let style = document.querySelector('#svelte-style');
		// If the style element does not exist, create a new one and set its ID
		if (!style) {
			style = document.createElement('style');
			style.id = 'svelte-style';
			head.appendChild(style);
		}
		// Set the innerHTML of the style element to the modified CSS code
		style.innerHTML = modifyCSS(css);
	}

	// Function: modifyCSS
	// Description: This function modifies the provided CSS code by applying specific replacements based on patterns.
	// It replaces pseudo-elements and pseudo-classes with modified versions, adjusts background and content URLs,
	// and modifies the values of certain CSS properties.
	// Parameters:
	//   - cssCode (string): The CSS code that needs to be modified.
	// Returns: string - The modified CSS code.
	function modifyCSS(cssCode) {
		// Regular expressions for different patterns in the CSS code
		const urlContentPattern = /(?:background(?:-image)?|content)\s*:\s*url\((.*?)\)/g;
		const positionPattern = /position\s*:\s*[^;]+\;/g;
		const cssBlockPattern = /{([^}]*)}/g;
		const transformPattern = /transform\s*:\s*([^;]+);/g;

		// Replace pseudo-elements and pseudo-classes with modified versions
		let newCSSCode = cssCode.replace(
			/(\w+)\s*::(after|before)|(\w+)\s*:(after|before)/g,
			(match, p1, p2, p3, p4) => {
				if (p1 && p2) {
					return p2 === 'before' ? `${p1}1::${p2}` : `${p1}2::${p2}`;
				} else if (p3 && p4) {
					return p3 === 'before' ? `${p3}1::${p4}` : `${p3}2::${p4}`;
				} else {
					return match;
				}
			}
		);

		// Replace background and content URLs with a common replacement
		newCSSCode = newCSSCode.replace(urlContentPattern, (match, url) => {
			const commonReplacement = `--modified: 'modified';\nbackground-image:url(${url});\nbackground-size: 120px 120px !important;\nbackground-repeat: no-repeat;\ndisplay: inline-block;\nwidth: 120px !important;\nheight: 120px !important;\ncontent:""`;
			return match.includes('background-image') || match.includes('content')
				? commonReplacement
				: match;
		});

		// Remove the position property from the CSS code
		newCSSCode = newCSSCode.replace(positionPattern, '');

		newCSSCode = newCSSCode.replace(cssBlockPattern, (match, cssBlock) => {
			if (cssBlock.includes('--modified:')) {
				const hasTransform = cssBlock.match(transformPattern);
				if (hasTransform) {
					// Excluding "scale" from the replacement
					const modifiedTransform = cssBlock.replace(transformPattern, (match, p1) => {
						const withoutScale = p1.replace(/scale\s*\([^)]*\)/g, '');
						return `transform: ${withoutScale};`;
					});
					return `{${modifiedTransform}}`;
				} else {
					return `{${cssBlock}}`;
				}
			} else {
				// Replace values bigger than 100px with 100px, and keep other values unchanged
				const modifiedBlock = cssBlock.replace(/(\d+px)/g, (match, p1) => {
					const value = parseInt(p1, 10);
					return value > 80 ? '80px' : p1;
				});
				return `{${modifiedBlock}}`;
			}
		});

		// Return the modified CSS code
		return newCSSCode;
	}

	/// Function: extractClassesAndRules
	// Description: This function extracts classes and their corresponding CSS rules from the provided CSS code.
	// It uses a regular expression to match CSS class declarations and iterates through the matches to collect class information.
	// Parameters:
	//   - cssCode (string): The CSS code from which classes and rules need to be extracted.
	// Returns: Array of Objects - An array of objects containing class information with 'originalClass', 'newClass', and 'css' properties.
	function extractClassesAndRules(cssCode) {
		const classAndRulesList = [];
		const classRegex = /\.([\w-][^{;]*?)\s*{([^}]*?)}/gs;

		let match;
		while ((match = classRegex.exec(cssCode))) {
			const className = match[1].replace(/::.*|:.*/g, '');
			const originalClassName = match[1].replace(/(1|2)(::|:).*/g, '');
			const cssRules = match[2];
			classAndRulesList.push({
				originalClass: originalClassName,
				newClass: className,
				css: cssRules
			});
		}

		return classAndRulesList;
	}

	// Function: extractTitleAndDescription
	// Description: This function extracts the title, description, and modified information from the provided CSS rules.
	// It uses regular expressions to match specific CSS properties ('--title', '--description', and '--modified') and extract their values.
	// Parameters:
	//   - cssRules (string): The CSS rules from which title, description, and modified information need to be extracted.
	// Returns: Object - An object containing 'title', 'description', and 'modified' properties.
	function extractTitleAndDescription(cssRules) {
		const titleRegex = /--title:\s*(['"])(.*?)\1\s*;/;
		const descriptionRegex = /--description:\s*(['"])(.*?)\1\s*;/;
		const modifiedRegex = /--modified:\s*'([^']+)'/;

		const titleMatch = cssRules.match(titleRegex);
		const descriptionMatch = cssRules.match(descriptionRegex);
		const modifiedMatch = cssRules.match(modifiedRegex);

		const title = titleMatch ? titleMatch[2] : 'No title...';
		const description = descriptionMatch ? descriptionMatch[2] : 'No description...';
		const modified = modifiedMatch ? modifiedMatch[1] : '';

		return { title, description, modified };
	}

	// Function: extractClassInfo
	// Description: This function processes the provided CSS code and extracts class information as JSON objects.
	// It uses the previously defined functions 'extractClassesAndRules' and 'extractTitleAndDescription' to obtain class details.
	// Parameters:
	//   - cssCode (string): The CSS code from which class information needs to be extracted.
	// Returns: Array of Objects - An array of objects containing class information with 'originalClass', 'newClass', 'title', 'description', and 'modified' properties.
	function extractClassInfo(cssCode) {
		const classAndRulesList = extractClassesAndRules(cssCode);
		const classInfoList = [];

		classAndRulesList.forEach(({ originalClass: originalClassName, newClass: className, css }) => {
			const { title, description, modified } = extractTitleAndDescription(css);
			classInfoList.push({
				originalClass: originalClassName,
				newClass: className,
				title,
				description,
				modified
			});
		});

		return classInfoList;
	}

	// Function: tabChange
	// Description: This function is triggered when a new tab is selected in the Tab component.
	// It updates the 'activeItem' variable to the newly selected tab and disposes the 'editor' if it exists.
	// If the selected tab is 'Code Editor', it calls the 'setupEditor' function to initialize the editor and 'tableauInit' to initialize Tableau settings.
	// Parameters:
	//   - e (CustomEvent): The custom event object containing the selected tab information.
	// Returns: void (Does not return anything)
	async function tabChange(e) {
		activeItem = e.detail;
	}

	// Function: copy
	// Description: This function copies the given 'text' to the clipboard using the 'copy' function from 'copy-to-clipboard' library.
	// It updates the 'tooltipText' variable to show a tooltip message indicating that the text has been copied.
	// Parameters:
	//   - text (string): The text to be copied to the clipboard.
	// Returns: void (Does not return anything)
	function copyClass(text) {
		copy(text);
		tooltipText = 'Copied: ' + text;
	}

	// Function: outFunc
	// Description: This function resets the 'tooltipText' variable to its default value when the mouse is moved out of the tooltip element.
	// Parameters: None
	// Returns: void (Does not return anything)
	function outFunc() {
		tooltipText = 'Copy to copy!';
	}
</script>

<Tab {navList} {activeItem} on:tabChange={tabChange}>
	{#if activeItem.name === 'Bookmarks'}
		<div class="sideTitle">
			<h1>Bookmarks</h1>
			<p>Take a look at the available elements to use on your dashboard.</p>
		</div>
		<div id="bookmarks">
			{#each classInfoList as elem, index (elem)}
				{#if index !== 0}
					<hr />
				{/if}

				<div class="flex elem-box">
					<div class="elem-class-box {elem.modified}">
						<div class="handleOverflow">
							<div class="{elem.newClass} elem-class" />
						</div>
					</div>

					<!-- svelte-ignore a11y-no-static-element-interactions -->
					<div class="class-info">
						<div class="class-names">
							<h2>{elem.title}</h2>

							<!-- svelte-ignore a11y-click-events-have-key-events -->
							<!-- svelte-ignore a11y-mouse-events-have-key-events -->
							<div
								class="copy tooltip fade"
								on:click={() => copyClass(elem.originalClass)}
								on:mouseout={outFunc}
							>
								<p>{elem.originalClass}</p>
								<span class="tooltiptext" id="myTooltip">{tooltipText}</span>
								<Fa icon={faCopy} size="15" color="#999" />
							</div>
						</div>
						<div class="elem-desc">
							<p>{elem.description}</p>
						</div>
					</div>
				</div>
			{/each}
		</div>
		<a href={template} target="_blank" rel="noopener noreferrer">
			<Button classes={'on'} text={'Download Template'} icon={faDownload} />
		</a>
	{:else if activeItem.name === 'Configuration'}
		<div class="sideTitle">
			<h1>Under Construction</h1>
			<p>This page will be available soon!</p>
		</div>
		<img src={Soon} alt="" srcset="" />
	{/if}
</Tab>

<style>
	/* Typography */
	h1 {
		font-weight: bold;
		font-size: 28px;
	}

	p {
		font-weight: 400;
		font-size: 14px;
		line-height: 24px;
		color: #565656;
	}

	/* Layout */
	hr {
		margin: 20px;
	}

	img {
		width: 100%;
		max-width: 450px;
	}

	.flex {
		display: flex;
		align-items: flex-start;
		padding: 0;
		width: 100%;
	}

	#bookmarks {
		overflow-y: auto;
	}

	.elem-box {
		background-color: #fafafa;
		padding: 30px 40px;
		display: flex;
		gap: 60px;
	}

	.elem-class-box {
		width: 120px;
		height: 120px;
		justify-content: center;
		display: flex;
	}

	.handleOverflow {
		overflow: hidden;
		width: max-content;
		border-radius: 10px;
	}

	.elem-class {
		width: 120px;
		height: 120px;
	}

	.class-info {
		display: flex;
		flex-direction: column;
		gap: 5px;
		width: 100%;
	}

	.elem-desc {
		max-height: 120px;
		overflow: auto;
	}

	.class-names,
	.copy {
		display: flex;
		justify-content: space-between;
		overflow: visible;
		gap: 15px;
		align-items: center;
	}

	.copy {
		cursor: pointer;
	}

	.class-names p {
		font-style: italic;
	}

	/* Tooltips */
	.tooltip {
		position: relative;
	}

	.tooltip .tooltiptext {
		visibility: hidden;
		background-color: #555;
		color: #fff;
		font-size: 12px;
		text-align: center;
		border-radius: 6px;
		padding: 5px 15px;
		position: absolute;
		z-index: 1;
		bottom: 150%;
		left: 50%;
		margin-left: -75px;
		opacity: 0;
		transition: opacity 0.3s;
		white-space: nowrap;
	}

	.tooltip .tooltiptext::after {
		content: '';
		position: absolute;
		top: 100%;
		left: 50%;
		margin-left: -5px;
		border-width: 5px;
		border-style: solid;
		border-color: #555 transparent transparent transparent;
	}

	.tooltip:hover .tooltiptext {
		visibility: visible;
		opacity: 1;
	}
</style>
