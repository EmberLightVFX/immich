<script lang="ts">
  import { timeBeforeShowLoadingSpinner } from '$lib/constants';
  import { handleError } from '$lib/utils/handle-error';
  import {
    createPerson,
    getAllPeople,
    reassignFaces,
    type AssetFaceUpdateItem,
    type PersonResponseDto,
  } from '@immich/sdk';
  import { Button } from '@immich/ui';
  import { mdiMerge, mdiPlus } from '@mdi/js';
  import { onMount, type Snippet } from 'svelte';
  import { t } from 'svelte-i18n';
  import { quintOut } from 'svelte/easing';
  import { fly } from 'svelte/transition';
  import ControlAppBar from '../shared-components/control-app-bar.svelte';
  import { NotificationType, notificationController } from '../shared-components/notification/notification';
  import FaceThumbnail from './face-thumbnail.svelte';
  import PeopleList from './people-list.svelte';

  interface Props {
    assetIds: string[];
    personAssets: PersonResponseDto;
    onConfirm: () => void;
    onClose: () => void;
    header?: Snippet;
    merge?: Snippet;
  }

  let { assetIds, personAssets, onConfirm, onClose, header, merge }: Props = $props();

  let people: PersonResponseDto[] = $state([]);
  let selectedPerson: PersonResponseDto | null = $state(null);
  let disableButtons = $state(false);
  let showLoadingSpinnerCreate = $state(false);
  let showLoadingSpinnerReassign = $state(false);
  let hasSelection = $state(false);
  let screenHeight: number = $state(0);

  let peopleToNotShow = $derived(selectedPerson ? [personAssets, selectedPerson] : [personAssets]);

  const selectedPeople: AssetFaceUpdateItem[] = [];

  for (const assetId of assetIds) {
    selectedPeople.push({ assetId, personId: personAssets.id });
  }

  onMount(async () => {
    const data = await getAllPeople({ withHidden: false });
    people = data.people;
  });

  const handleSelectedPerson = (person: PersonResponseDto) => {
    if (selectedPerson && selectedPerson.id === person.id) {
      handleRemoveSelectedPerson();
      return;
    }
    selectedPerson = person;
    hasSelection = true;
  };

  const handleRemoveSelectedPerson = () => {
    selectedPerson = null;
    hasSelection = false;
  };

  const handleCreate = async () => {
    const timeout = setTimeout(() => (showLoadingSpinnerCreate = true), timeBeforeShowLoadingSpinner);

    try {
      disableButtons = true;
      const data = await createPerson({ personCreateDto: {} });
      await reassignFaces({ id: data.id, assetFaceUpdateDto: { data: selectedPeople } });

      notificationController.show({
        message: $t('reassigned_assets_to_new_person', { values: { count: assetIds.length } }),
        type: NotificationType.Info,
      });
    } catch (error) {
      handleError(error, $t('errors.unable_to_reassign_assets_new_person'));
    } finally {
      clearTimeout(timeout);
    }

    showLoadingSpinnerCreate = false;
    onConfirm();
  };

  const handleReassign = async () => {
    const timeout = setTimeout(() => (showLoadingSpinnerReassign = true), timeBeforeShowLoadingSpinner);
    try {
      disableButtons = true;
      if (selectedPerson) {
        await reassignFaces({ id: selectedPerson.id, assetFaceUpdateDto: { data: selectedPeople } });
        notificationController.show({
          message: $t('reassigned_assets_to_existing_person', {
            values: { count: assetIds.length, name: selectedPerson.name || null },
          }),
          type: NotificationType.Info,
        });
      }
    } catch (error) {
      handleError(
        error,
        $t('errors.unable_to_reassign_assets_existing_person', { values: { name: selectedPerson?.name || null } }),
      );
    } finally {
      clearTimeout(timeout);
    }

    showLoadingSpinnerReassign = false;
    onConfirm();
  };
</script>

<svelte:window bind:innerHeight={screenHeight} />

<section
  transition:fly={{ y: 500, duration: 100, easing: quintOut }}
  class="absolute start-0 top-0 h-full w-full bg-light"
>
  <ControlAppBar {onClose}>
    {#snippet leading()}
      {@render header?.()}
      <div></div>
    {/snippet}
    {#snippet trailing()}
      <div class="flex gap-4">
        <Button
          shape="round"
          title={$t('create_new_person_hint')}
          leadingIcon={mdiPlus}
          loading={showLoadingSpinnerCreate}
          size="small"
          disabled={disableButtons || hasSelection}
          onclick={handleCreate}
        >
          {$t('create_new_person')}</Button
        >
        <Button
          size="small"
          shape="round"
          title={$t('reassing_hint')}
          leadingIcon={mdiMerge}
          loading={showLoadingSpinnerReassign}
          disabled={disableButtons || !hasSelection}
          onclick={handleReassign}
        >
          {$t('reassign')}
        </Button>
      </div>
    {/snippet}
  </ControlAppBar>
  {@render merge?.()}
  <section class="px-[70px] pt-[100px]">
    <section id="merge-face-selector relative">
      {#if selectedPerson !== null}
        <div class="mb-10 h-[200px] place-content-center place-items-center">
          <p class="mb-4 text-center uppercase dark:text-white">Choose matching faces to re assign</p>

          <div class="grid grid-flow-col-dense place-content-center place-items-center gap-4">
            <FaceThumbnail
              person={selectedPerson}
              border
              circle
              selectable
              thumbnailSize={180}
              onClick={handleRemoveSelectedPerson}
            />
          </div>
        </div>
      {/if}
      <PeopleList {people} {peopleToNotShow} {screenHeight} onSelect={handleSelectedPerson} />
    </section>
  </section>
</section>
