<script>
  export let ticketId;
  export let eventId;
  export let title;
  export let text;
  export let isOwedShirt = false;

  import { getClient } from '@urql/svelte';
  import { createEventDispatcher } from 'svelte';
  import { Circle3 } from 'svelte-loading-spinners';
  import Checkbox from 'svelte-checkbox';

  import Panel from '../../../components/numberPad/Panel.svelte';
  import { Shell } from '../../../elements/buttons';
  import checkinMutationApi from '../../../dataSources/api.that.tech/checkin/mutations';

  const dispatch = createEventDispatcher();
  const { setPartnerPin, revertCheckIn, setReceivedSwag } = checkinMutationApi(
    getClient(),
  );

  let pinNumber = '';
  let waiting = false;
  let checkInError = false;
  let checkInErroMessage;
  let setPin = false;

  async function handleResetPin() {
    waiting = true;
    woopra.track('event_inperson_edit_checkin');

    const { result, message } = await setPartnerPin(
      eventId,
      ticketId,
      pinNumber,
    );

    waiting = false;

    if (result) {
      dispatch('checkinUpdated');
    } else {
      checkInError = true;
      checkInErroMessage = message;
    }
  }

  async function handleRevertCheckIn() {
    waiting = true;
    woopra.track('event_inperson_edit_checkin');

    const { result, message } = await revertCheckIn(eventId, ticketId);

    waiting = false;

    if (result) {
      dispatch('checkinUpdated');
    } else {
      checkInError = true;
      checkInErroMessage = message;
    }
  }

  function tryAgain() {
    checkInError = false;
    checkInErroMessage = undefined;
  }

  async function handleOweSwag() {
    waiting = true;
    woopra.track('event_inperson_edit_checkin');

    const { result, message } = await setReceivedSwag(
      eventId,
      ticketId,
      !isOwedShirt,
    );

    waiting = false;

    if (result) {
      dispatch('checkinUpdated');
    } else {
      checkInError = false;
      checkInErroMessage = message;
    }
  }
</script>

<div
  class="z-10 fixed bottom-0 inset-x-0 px-4 pb-6 sm:inset-0 sm:p-0 sm:flex sm:items-center sm:justify-center">
  <div class="fixed inset-0 transition-opacity">
    <div class="absolute inset-0 bg-gray-500 opacity-75"></div>
  </div>

  <div
    class="bg-white rounded-lg px-4 pt-5 pb-4 overflow-hidden shadow-xl transform transition-all sm:max-w-sm sm:w-full sm:p-6">
    <div class="hidden sm:block absolute top-0 right-0 pt-4 pr-4">
      <button
        type="button"
        class=" rounded-md text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-thatBlue-500"
        on:click="{() => dispatch('close')}">
        <span class="sr-only">Close</span>
        <svg
          class="h-6 w-6"
          xmlns="http://www.w3.org/2000/svg"
          fill="none"
          viewBox="0 0 24 24"
          stroke="currentColor"
          aria-hidden="true">
          <path
            stroke-linecap="round"
            stroke-linejoin="round"
            stroke-width="2"
            d="M6 18L18 6M6 6l12 12"></path>
        </svg>
      </button>
    </div>

    <div>
      <div class="mt-4 text-center sm:mt-5">
        <h1
          class="text-xl leading-6 font-bold text-gray-900"
          id="modal-headline">
          {title}
        </h1>
        <div class="mt-2">
          <p class="text-md leading-5 text-gray-500">{text}</p>
        </div>

        {#if waiting}
          <div class="relative my-6 flex flex-col items-center space-y-6">
            <Circle3
              size="{'60'}"
              unit="{'px'}"
              ballTopLeft="#f74646"
              ballTopRight="#ff834d"
              ballBottomLeft="#26529A"
              ballBottomRight="#555555" />

            <div class="mt-2">
              <p class="text-md leading-5 text-gray-500">
                Invoking api and making data.
              </p>
            </div>
          </div>
        {:else if checkInError}
          <div class="relative my-6 flex flex-col space-y-6">
            <div class="w-full mt-2">
              <p class="text-md leading-5 text-gray-500">
                {checkInErroMessage}
              </p>
            </div>

            <div class="flex">
              <Shell>
                <button
                  class="w-full py-4 text-sm leading-5 font-medium"
                  on:click="{() => tryAgain()}">
                  <span class="text-lg">Retry</span>
                </button>
              </Shell>
            </div>
          </div>
        {:else if !setPin}
          <div class="relative mt-6 flex flex-col space-y-6">
            <div class="flex">
              <Shell>
                <button
                  class="w-full py-4 text-sm leading-5 font-medium"
                  on:click="{() => handleRevertCheckIn()}">
                  <span class="text-lg">Revert CheckIn</span>
                </button>
              </Shell>
            </div>

            <div class="flex">
              <Shell>
                <button
                  class="w-full py-4 text-sm leading-5 font-medium"
                  on:click="{() => (setPin = true)}">
                  <span class="text-lg">Reset Pin</span>
                </button>
              </Shell>
            </div>

            <div class="flex space-x-4 items-center">
              <div>
                <Checkbox
                  name="isOwedShirt"
                  bind:checked="{isOwedShirt}"
                  size="3.5rem"
                  class="flex-none" />
              </div>
              <Shell>
                <button
                  class="w-full py-4 text-sm leading-5 font-medium"
                  on:click="{() => handleOweSwag()}">
                  <span class="text-lg">Owe Shirt</span>
                </button>
              </Shell>
            </div>
          </div>
        {:else}
          <div class="relative mt-6 flex flex-col space-y-6">
            <div>
              <Panel
                on:digit="{event => {
                  pinNumber = pinNumber.concat(event.detail);
                }}"
                on:backspace="{event => {
                  pinNumber = pinNumber.slice(0, -1);
                }}" />
            </div>

            <div
              class="relative border border-gray-300 rounded-md px-3 py-2 shadow-sm focus-within:ring-1 focus-within:ring-thatBlue-800 focus-within:border-thatBlue-800">
              <input
                type="text"
                name="pinNumber"
                bind:value="{pinNumber}"
                class="block w-full border-0 p-0 text-gray-900 placeholder-gray-400 focus:ring-0 sm:text-sm"
                placeholder="1234" />
            </div>

            <div class="flex">
              <Shell>
                <button
                  class="w-full py-4 text-sm leading-5 font-medium"
                  on:click="{() => handleResetPin()}">
                  <span class="text-lg">Reset Pin</span>
                </button>
              </Shell>
            </div>
          </div>
        {/if}
      </div>
    </div>
  </div>
</div>
