<script>
	import '../lib/css/tab-css.css';
	import '../lib/css/bootstrap.min.css';
	import { onMount, onDestroy } from 'svelte';

	async function init(tableauExt) {
		//clean up any divs from the last initialization
		document.querySelector('body').innerHTML = '';
		tableau.extensions
			.setClickThroughAsync(true)
			.then(() => {
				let dashboard = tableauExt.dashboardContent.dashboard;
				//Loop through the Objects on the Dashboard and render the HTML Objects
				dashboard.objects.forEach((obj) => {
					render(obj);
				});
			})
			.catch((error) => {
				// Can throw an error if called from a dialog or on Tableau Desktop
				console.error(error.message);
			});
	}

	// Function to calculate margins from object classes
	function getMarginFromObjClasses(objClasses) {
		const margin = [0, 0, 0, 0];
		if (!objClasses) return margin;

		const classNames = objClasses.split(/\s+/);
		classNames.reverse();
		const marginClass = classNames.find((cl) => cl.startsWith('margin-'));
		if (!marginClass) return margin;

		const marginValues = marginClass
			.split('-')
			.slice(1)
			.map((v) => parseInt(v));
		if (marginValues.length === 1) {
			const [all] = marginValues;
			return [all, all, all, all];
		}

		if (marginValues.length === 2) {
			const [vertical, horizontal] = marginValues;
			return [vertical, horizontal, vertical, horizontal];
		}

		if (marginValues.length === 3) {
			const [top, horizontal, bottom] = marginValues;
			return [top, horizontal, bottom, horizontal];
		}

		if (marginValues.length === 4) {
			return marginValues;
		}

		return margin;
	}

	async function render(obj) {
		let objNameAndClasses = obj.name.split('|');
		//Parse the Name and Classes from the Object Name
		let objId = objNameAndClasses[0];
		let objClasses;
		//Check if there are classes on the object
		if (objNameAndClasses.length > 1) {
			objClasses = objNameAndClasses[1];
		}
		//Create the initial object with CSS Props

		// we need to check for padding classes first, as they must be handled via positioning
		const margin = getMarginFromObjClasses(objClasses);

		//Here we set the CSS props to match the location of the objects on the Dashboard
		let props = {
			id: `${objId}`,
			style: `
                position: absolute;
                top: ${parseInt(obj.position.y) + margin[0]}px;
                left: ${parseInt(obj.position.x) + margin[3]}px;
                width: ${parseInt(obj.size.width) - margin[1] - margin[3]}px;
                height: ${parseInt(obj.size.height) - margin[0] - margin[2]}px;
                `
		};

		const $div = document.createElement('div');
		$div.id = props.id;
		$div.style.cssText = props.style;
		if (objClasses) {
			// Split the objClasses string into individual class names
			const classesArr = objClasses.split(/\s+/);

			// Add each class separately to the classList
			classesArr.forEach((className) => {
				if (className) {
					$div.classList.add(className);
				}
			});
		}
		document.body.appendChild($div);
	}

	let resizeEventHandler; // Declare the event handler variable at the top level

	onMount(async () => {
		let tableauExt = window.tableau.extensions;
		if (!tableauExt) return;

		tableauExt.initializeAsync({ configure: configure }).then(
			async function () {
				init(tableauExt);
				//Register an event handler for Dashboard Object resize
				//Supports automatic sized dashboards and reloads
				resizeEventHandler = tableauExt.dashboardContent.dashboard.addEventListener(
					tableau.TableauEventType.DashboardLayoutChanged,
					() => init(tableauExt)
				);
			},
			(err) => {
				console.log('Broken');
			}
		);

		function configure() {
			const popupUrl = `${window.location.origin}/extensions/ultimate-tab-css-business/popup`;
			const defaultIntervalInMin = '5';
			tableauExt.ui
				.displayDialogAsync(popupUrl, defaultIntervalInMin, {
					height: 800,
					width: 800
				})
				.then((closePayload) => {});
		}
	});

	onDestroy(() => {
		// Unregister the event handler when the component is destroyed
		if (resizeEventHandler) {
			window.tableau.extensions.dashboardContent.dashboard.removeEventListener(
				tableau.TableauEventType.DashboardLayoutChanged,
				resizeEventHandler
			);
		}
	});
</script>

<style src="./style.css"></style>
