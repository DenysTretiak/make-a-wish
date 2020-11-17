<script>
  import formatMessage from "format-message";
  import translations from "../locales";
  import Spinner from "./Spinner.svelte";
  import Switch from './Switch.svelte';
  import SearchField from './SearchField.svelte';
  import Tags from './Tags.svelte';
  import categories from '../data/categories';
  import { profile } from "../stores";
  export let handleDismiss;
  export let handleSuccess;
  export let message = "";
  export let showMagicForm = false;
  export let showSuccess = false;
  let formRef;
  let fileInputRef;
  let lang;
  let loading = false;
  let acceptTerms = false;
  let query = '';
  let shownCategoriesCount = 6;
  let totalCount = categories.length;
  let activeValues = [];

  let localCategories = categories;

  $: shownCategories = localCategories.slice(0, shownCategoriesCount);
  $: filteredList = shownCategories.filter(category => category.name.toLocaleLowerCase().indexOf(query.toLocaleLowerCase()) !== -1);
  $: isListFiltered = filteredList.length !== shownCategories.length;
  $: lang = $profile.lang || "ru";

  $: {
    formatMessage.setup({
      missingTranslation: "ignore",
      translations,
      locale: lang
    });
  }

  const handleMore = () => {
    shownCategoriesCount += 7;
  }

  const handleTagClick = (item, index) => {
    activeValues = [item, ...activeValues];

    if (isListFiltered) { 
      filteredList.splice(index, 1);
      filteredList = [item, ...filteredList];
    } else {
      localCategories.splice(index, 1);
      localCategories = [item, ...localCategories];
    }

  }

  const handleTagDismiss = (item, index) => {
    activeValues.splice(index, 1);
    activeValues = [...activeValues];

    if (isListFiltered) {
      filteredList.splice(index, 1);
      filteredList.splice(activeValues.length, 0, item);
      filteredList = [...filteredList];
    } else {
      localCategories.splice(index, 1);
      localCategories.splice(activeValues.length, 0, item);
      localCategories = [...localCategories];
    }
  }

  const rebuildList = () => {
    activeValues.forEach(({value}) => {
      let activeFilter = filteredList.find(filter => filter.value === value);

      if (activeFilter) {
        if (isListFiltered) {
          let index = filteredList.indexOf(activeFilter);

          filteredList.splice(index, 1);
          filteredList = [activeFilter, ...filteredList];
        } else {
          let index = localCategories.indexOf(activeFilter);

          localCategories.splice(index, 1);
          localCategories = [activeFilter, ...localCategories];
        }
      }
    })
  }

  const handleKeyup = () => {
    rebuildList();
  }

  const handleReset = () => {
    query = ''
    
    setTimeout(() => {
      rebuildList();
    }, 0)
  }

  const _handleSubmit = e => {
    const options = activeValues.map(({value}) => value);

    fetch('https://djinni.co/api/domagic', {
      method: 'POST',
      mode: 'no-cors',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        options,
        spell: message
      })
    })
    .then(res => showSuccess = res.ok);

    e.preventDefault();
    if (handleSuccess) handleSuccess();
  };
</script>

<style lang="scss">
  @import "../styles/theme";

  .job-chat-wrapper {
    display: block;
    position: relative;
    width: 100%;

    .loader-wrapper {
      display: flex;
      position: absolute;
      width: 100%;
      height: 100%;
      justify-content: center;
      align-items: center;
      top: 0;
      margin-left: -25px;
    }

    .faded {
      opacity: 0.25;
    }

    .hidden {
      opacity: 0;
    }

    & .job-message-wrapper {
      width: 100%;
      height: auto;

      & .message-bubble {
        width: 100%;
        height: 100%;
        background: white;
        border-radius: 12px;
        // border-bottom-right-radius: 0;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.22);
        padding: 20px 25px;
        padding-bottom: 10px;

        & textarea {
          font-size: 15px;
          height: 170px;

          &::placeholder {
            color: $lightText;
            opacity: 0.7;
          }
        }

        & .attachment {
          display: block;
          width: 100%;
          border-radius: 5px;
          background: $lightestFill;
          padding: 10px;
          color: $lightText;
          font-size: 13px;
          border: 1px solid $lightestFill;
          // padding-right: 30px;

          & .clickable {
            width: 100%;
            height: 100%;
            cursor: pointer;
            display: block;

            &:hover {
              color: $darkText;
            }
          }

          & a {
            text-decoration: none;
            color: $lightText;

            &:hover {
              text-decoration: underline;
            }
          }

          & i {
            display: inline-block;
            margin-right: 5px;
          }

          & i.trailing {
            float: right;
            padding: 10px;
            cursor: pointer;
            margin-top: -7px;
            margin-right: -10px;

            &:hover {
              color: $darkText;

              & + i + span {
                text-decoration: line-through;
              }
            }
          }
        }
      }
    }

    & .message-actions {
      width: 100%;
      margin: 20px 0;
      text-align: right;

      & button {
        background: $accent;
        color: white;
        // box-shadow: 0 2px 1px rgba(0, 0, 0, 0.2);

        &:hover {
          background: $accentHover;
          box-shadow: none;
        }
        &:disabled {
          color: rgba(0, 0, 0, 0.3);
          background: rgba(0, 0, 0, 0.1);
          box-shadow: none;
        }
      }

      & span {
        margin-left: 20px;

        &.text-button {
          color: black;
          margin-left: 0;
          float: left;
        }
      }
    }

    & p.job-message-hint {
      font-size: 13px;
      color: $black;
      text-align: right;
      margin: 0;

      & a {
        color: $black;
        font-weight: 700;

        &:hover {
          text-decoration: underline;
        }
      }
    }
  }

  .hidden {
    visibility: hidden;
  }

  @media (max-width: $small) {
    .job-chat-wrapper .message-actions span {
      width: 100%;
      margin: 10px 0;

      &.text-button {
        float: none;
      }
    }

    .job-chat-wrapper .message-actions {
      margin-bottom: 0;
    }

    .job-chat-wrapper .message-actions button {
      display: block;
      width: 100%;
    }

    .job-chat-wrapper p.job-message-hint {
      text-align: center;
    }

    .job-chat-wrapper .message-actions,
    .job-chat-wrapper p.job-message-hint {
      padding-right: 0;
    }
  }

  // @media (max-width: $tiny) {
  //   .job-chat-wrapper .message-actions {
  //     padding-right: 0;
  //   }
  // }
</style>

<section class="job-chat-wrapper" id="chat-wrapper">
  <form bind:this={formRef} id="applyForm">
    <div class="job-message-wrapper">
      {#if showMagicForm}
        <SearchField bind:query={query} handleKeyup={handleKeyup} handleReset={handleReset}/>
        <Tags 
          items={filteredList}
          handleMore={isListFiltered ? null : handleMore}
          totalCount={totalCount}
          activeValues={activeValues}
          handleClick={handleTagClick}
          handleDismiss={handleTagDismiss}/>
      {/if}
      
      <div class="message-bubble">
        <textarea
          id="message"
          name="message"
          placeholder = {showMagicForm ? 'Type a spell…' : null}
          class:faded={loading}
          bind:value={message} />
        {#if !showMagicForm}
          <p class="attachment" class:faded={loading}>
            {#if $profile.cv_name}
              <i
                class="icon-trash-empty trailing"
                on:click={() => {
                  $profile.cv_url = null;
                  $profile.cv_name = null;
                  fileInputRef.value = null;
                }} />
              <i class="icon-thumbs-up" />
              <span>
                {#if $profile.cv_url}
                  <a href={$profile.cv_url}>{$profile.cv_name}</a>
                {:else}{$profile.cv_name}{/if}
              </span>
            {:else}
              <span
                class="clickable"
                on:click={() => {
                  fileInputRef.click();
                }}>
                <i class="icon-doc-text" />
                {formatMessage('Upload resume')}
              </span>
            {/if}
          </p>
        {/if}
        {#if loading}
          <div class="loader-wrapper">
            <Spinner />
          </div>
        {/if}
      </div>
    </div>
    {#if showMagicForm}
      <Switch bind:checked={acceptTerms}></Switch>
    {/if}
    <div class="message-actions" class:hidden={loading}>
      <button name="job_apply" id="job_apply" on:click={_handleSubmit} disabled={showMagicForm ? (!message || !acceptTerms) : false}>
        {showMagicForm ? formatMessage('Save') : formatMessage('Open contacts and send')}
      </button>
      <span class="text-button" on:click={handleDismiss}>
        {formatMessage('Cancel')}
      </span>
    </div>
    <p class="job-message-hint">
      {formatMessage('What will the employer see?')}
      <br />
      {formatMessage('Set it')}
      <a
        href={$profile.usertype == 'recruiter' ? '/home/profile' : '/my/account'}>
        {formatMessage('in settings')}
      </a>
    </p>
    <input
      type="file"
      class="hidden"
      name="cv_direct"
      on:change={() => {
        $profile.cv_name = fileInputRef && fileInputRef.files && fileInputRef.files[0] && fileInputRef.files[0].name;
        $profile.cv_url = '';
      }}
      bind:this={fileInputRef} />
    <input
      type="hidden"
      name="cv_clear"
      value={$profile.cv_name ? '' : '(hidden)'} />
    <input type="hidden" name="cv_url" value={$profile.cv_url} />
    <!--<input type="hidden" name="csrfmiddlewaretoken" value={token} /> -->
  </form>
</section>
