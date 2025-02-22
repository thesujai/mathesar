<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  import { _ } from 'svelte-i18n';
  import {
    Checkbox,
    ControlledModal,
    LabeledInput,
    ModalController,
    CancelOrProceedButtonPair,
  } from '@mathesar-component-library';
  import DocsLink from '@mathesar/components/DocsLink.svelte';
  import Identifier from '@mathesar/components/Identifier.svelte';
  import RichText from '@mathesar/components/rich-text/RichText.svelte';
  import WarningBox from '@mathesar/components/message-boxes/WarningBox.svelte';
  import type { Connection } from '@mathesar/api/connections';
  import {
    connectionHasUniqueDatabaseReference,
    connectionsStore,
  } from '@mathesar/stores/databases';
  import { toast } from '@mathesar/stores/toast';
  import { getErrorMessage } from '@mathesar/utils/errors';

  const { connections } = connectionsStore;
  const dispatch = createEventDispatcher<{ delete: undefined }>();

  export let controller: ModalController;
  export let connection: Connection;

  $: hasUniqueDatabaseReference = connectionHasUniqueDatabaseReference(
    connection,
    $connections.values(),
  );

  let deleteMathesarSchemas = false;

  async function disconnect() {
    try {
      await connectionsStore.deleteConnection(
        connection.id,
        deleteMathesarSchemas,
      );
      toast.success($_('connection_deleted_successfully'));
      controller.close();
      dispatch('delete');
    } catch (e) {
      toast.error(getErrorMessage(e));
    }
  }

  function cancel() {
    controller.close();
    deleteMathesarSchemas = false;
  }
</script>

<ControlledModal {controller}>
  <svelte:fragment slot="title">
    <RichText text={$_('delete_connection_with_name')} let:slotName>
      {#if slotName === 'connectionName'}
        <Identifier>{connection.nickname}</Identifier>
      {/if}
    </RichText>
  </svelte:fragment>

  <p>{$_('action_cannot_be_undone')}</p>
  <p>
    {$_('delete_connection_info')}
  </p>
  <p>
    {$_('delete_connection_db_delete_info')}
  </p>

  {#if hasUniqueDatabaseReference}
    <p>
      <LabeledInput
        layout="inline-input-first"
        label={$_('delete_associated_mathesar_schemas_help')}
      >
        <Checkbox bind:checked={deleteMathesarSchemas} />
      </LabeledInput>
    </p>

    <p>
      <WarningBox>
        {$_('using_custom_types')}
        <DocsLink path="/">
          {$_('learn_implications_deleting_mathesar_schemas')}
        </DocsLink>
      </WarningBox>
    </p>
  {/if}

  <CancelOrProceedButtonPair
    proceedButton={{
      icon: undefined,
      label: $_('delete_connection'),
    }}
    onCancel={cancel}
    onProceed={disconnect}
    slot="footer"
  />
</ControlledModal>
