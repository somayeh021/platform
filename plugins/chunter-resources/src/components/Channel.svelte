<!--
// Copyright © 2023 Hardcore Engineering Inc.
//
// Licensed under the Eclipse Public License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License. You may
// obtain a copy of the License at https://www.eclipse.org/legal/epl-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
//
// See the License for the specific language governing permissions and
// limitations under the License.
-->
<script lang="ts">
  import { Class, Doc, Ref, Timestamp } from '@hcengineering/core'
  import { DocNotifyContext } from '@hcengineering/notification'
  import activity, { ActivityMessage, ActivityMessagesFilter } from '@hcengineering/activity'
  import { getClient } from '@hcengineering/presentation'
  import { getMessageFromLoc } from '@hcengineering/activity-resources'
  import { location as locationStore } from '@hcengineering/ui'

  import chunter from '../plugin'
  import ChannelScrollView from './ChannelScrollView.svelte'
  import { ChannelDataProvider } from '../channelDataProvider'

  export let context: DocNotifyContext
  export let object: Doc | undefined
  export let filters: Ref<ActivityMessagesFilter>[] = []

  const client = getClient()
  const hierarchy = client.getHierarchy()

  let dataProvider: ChannelDataProvider | undefined
  let selectedMessageId: Ref<ActivityMessage> | undefined = undefined

  locationStore.subscribe((newLocation) => {
    selectedMessageId = getMessageFromLoc(newLocation)
  })

  $: isDocChannel = !hierarchy.isDerived(context.attachedToClass, chunter.class.ChunterSpace)
  $: _class = isDocChannel ? activity.class.ActivityMessage : chunter.class.ChatMessage
  $: collection = isDocChannel ? 'comments' : 'messages'

  $: updateDataProvider(context.attachedTo, _class, context.lastViewedTimestamp, selectedMessageId)

  function updateDataProvider (
    attachedTo: Ref<Doc>,
    _class: Ref<Class<ActivityMessage>>,
    lastViewedTimestamp?: Timestamp,
    selectedMessageId?: Ref<ActivityMessage>
  ) {
    if (dataProvider === undefined) {
      // For now loading all messages for documents with activity. Need to correct handle aggregation with pagination.
      // Perhaps we should load all activity messages once, and keep loading in chunks only for ChatMessages then merge them correctly with activity messages
      const loadAll = isDocChannel
      dataProvider = new ChannelDataProvider(attachedTo, _class, lastViewedTimestamp, selectedMessageId, loadAll)
    }
  }
</script>

{#if dataProvider}
  <ChannelScrollView
    objectId={context.attachedTo}
    objectClass={context.attachedToClass}
    {object}
    skipLabels={!isDocChannel}
    selectedFilters={filters}
    startFromBottom
    {selectedMessageId}
    {collection}
    provider={dataProvider}
    loadMoreAllowed={!isDocChannel}
  />
{/if}
