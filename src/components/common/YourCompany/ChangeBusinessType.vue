<template>
  <div id="change-business-type">
    <v-row no-gutters>
      <!-- Row Title -->
      <v-col cols="3">
        <label :class="{'error-text': invalidSection}"><strong>Business Type</strong></label>
        <v-col
          md="1"
          class="pa-0"
        >
          <v-chip
            v-if="hasBusinessTypeChanged"
            x-small
            label
            color="primary"
            text-color="white"
          >
            {{ getEditedLabel }}
          </v-chip>
        </v-col>
      </v-col>

      <!-- Display Mode -->
      <v-col
        v-if="!isEditingType"
        cols="7"
      >
        <span
          class="info-text"
          :class="{ 'has-conflict': isConflictingLegalType && isNewName}"
        >
          {{ GetCorpFullDescription(getEntityType) }}
        </span>

        <!-- Type change tooltip -->
        <v-tooltip
          v-if="typeChangeInfo"
          top
          content-class="top-tooltip"
          transition="fade-transition"
          nudge-right="3"
        >
          <template #activator="{ on }">
            <v-icon
              class="info-icon"
              v-on="on"
            >
              mdi-information-outline
            </v-icon>
          </template>
          <span>{{ typeChangeInfo }}</span>
        </v-tooltip>

        <!-- Type mismatch tooltip -->
        <v-tooltip
          v-if="isConflictingLegalType && isNewName"
          top
          content-class="top-tooltip"
          transition="fade-transition"
          nudge-right="3"
        >
          <template #activator="{ on }">
            <v-icon
              color="error"
              small
              v-on="on"
            >
              mdi-alert
            </v-icon>
          </template>
          <span>
            Business Types do not match. The Name Request type must match the business type before you can continue.
          </span>
        </v-tooltip>

        <template v-if="hasBusinessTypeChanged">
          <p class="subtitle mt-2 pt-2">
            {{ updatedArticleTitle }}
          </p>
          <div class="confirmed-msg d-flex">
            <v-icon
              color="success"
              class="confirmed-icon d-block"
            >
              mdi-check
            </v-icon>
            <span class="info-text text-body-3 confirmed-icon ml-2 d-block">
              {{ updatedArticleInfo }}
            </span>
          </div>
        </template>
      </v-col>

      <!-- Editing Mode -->
      <v-col
        v-if="isEditingType"
        cols="9"
        class="pr-4"
      >
        <v-select
          id="business-type-selector"
          v-model="selectedEntityType"
          :class="{ 'disabled-select': isEntityTypeChangedByName }"
          :disabled="isEntityTypeChangedByName"
          :items="entityTypeOptions"
          :hint="isEntityTypeChangedByName ? '' : 'Select a New Business Type'"
          persistent-hint
          filled
        >
          <template #item="data">
            <span class="list-item">{{ data.item.text }}</span>
          </template>
        </v-select>
        <template v-if="isEntityTypeChangedByName && hasBusinessTypeChanged">
          <!-- Reminder: Type changed based on name request -->
          <v-row
            no-gutters
          >
            <v-col cols="auto">
              <v-icon
                class="pr-2"
                color="primary"
              >
                mdi-alert-circle-outline
              </v-icon>
            </v-col>

            <v-col>
              <div class="info-text">
                We have changed your business type based on the Name Request you entered.
              </div>
            </v-col>
          </v-row>
        </template>

        <div
          v-if="minimumThreeDirectorError"
          id="minimum-three-director-error"
          class="my-6"
        >
          <p class="error-text">
            The business type cannot be changed. A Community Contribution Company requires a minimum of three directors.
          </p>
        </div>

        <div
          v-if="nameRequestRequiredError"
          id="name-request-required-error"
          class="my-6"
        >
          <p class="error-text">
            An alteration of business type Name Request is required to make this change. After the name is approved,
            you can then select 'change' beside the company name to update it.
          </p>
        </div>

        <div class="my-6">
          <p class="info-text">
            Businesses can only be altered to specific types. If the business type you want is
            not listed, contact BC Registry staff:
          </p>
        </div>

        <!-- BC Registry Contacts -->
        <BcRegContacts :direction="'col'" />

        <BcRegEntityDetails
          :isBenefit="isBenefit"
          :isUnlimitedLiability="isUnlimitedLiability"
          :isCommunityContribution="isCommunityContribution"
          :isBcLimited="isBcLimited"
          :selectedEntityType="selectedEntityType"
          :confirmArticles="confirmArticles"
          @update:confirmArticles="confirmArticles = $event"
        />

        <!-- Done Actions -->
        <div class="action-btns">
          <v-btn
            id="done-btn"
            large
            color="primary"
            :disabled="disableDoneButton"
            @click="submitTypeChange()"
          >
            <span>Done</span>
          </v-btn>

          <v-btn
            id="cancel-btn"
            large
            outlined
            color="primary"
            @click="isEditingType = false"
          >
            <span>Cancel</span>
          </v-btn>
        </div>
      </v-col>

      <!-- Edit Actions -->
      <v-col
        v-if="!isEditingType"
        cols="2"
        class="mt-n2"
      >
        <div
          v-if="!isEntityTypeChangedByName"
          class="actions mr-4"
        >
          <v-btn
            v-if="hasBusinessTypeChanged"
            id="btn-undo-business-type"
            text
            color="primary"
            class="undo-action"
            @click="resetType()"
          >
            <v-icon small>
              mdi-undo
            </v-icon>
            <span>Undo</span>
          </v-btn>
          <v-btn
            v-else-if="enableEditButton"
            id="btn-correct-business-type"
            text
            color="primary"
            @click="isEditingType = true"
          >
            <v-icon small>
              mdi-pencil
            </v-icon>
            <span>{{ getEditLabel }}</span>
          </v-btn>
          <span
            v-if="hasBusinessTypeChanged"
            class="more-actions"
          >
            <v-menu
              v-model="dropdown"
              offset-y
              left
              nudge-bottom="4"
            >
              <template #activator="{ on }">
                <v-btn
                  id="btn-more-actions"
                  text
                  small
                  color="primary"
                  v-on="on"
                >
                  <v-icon>{{ dropdown ? 'mdi-menu-up' : 'mdi-menu-down' }}</v-icon>
                </v-btn>
              </template>
              <v-list>
                <v-list-item
                  id="btn-more-actions-edit"
                  class="v-list-item"
                  @click="isEditingType = true; dropdown = false"
                >
                  <v-list-item-subtitle>
                    <v-icon
                      small
                      color="primary"
                    >mdi-pencil</v-icon>
                    <span class="drop-down-action ml-1">Change</span>
                  </v-list-item-subtitle>
                </v-list-item>
              </v-list>
            </v-menu>
          </span>
        </div>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import { Component, Emit, Mixins, Prop, Watch } from 'vue-property-decorator'
import { Action, Getter } from 'pinia-class'
import BcRegEntityDetails from '@/components/Alteration/BcRegEntityDetails.vue'
import { BcRegContacts } from '@/components/common/'
import { CommonMixin } from '@/mixins/'
import { CorpTypeCd, GetCorpFullDescription } from '@bcrs-shared-components/corp-type-module/'
import { EntitySnapshotIF, EntityTypeOption, NameRequestIF, ResourceIF } from '@/interfaces/'
import { GetFeatureFlag, ResourceUtilities } from '@/utils'
import { useStore } from '@/store/store'

@Component({
  components: {
    BcRegContacts,
    BcRegEntityDetails
  }
})
export default class ChangeBusinessType extends Mixins(CommonMixin) {
  // for template
  readonly GetCorpFullDescription = GetCorpFullDescription

  @Prop({ default: false }) readonly invalidSection!: boolean

  // Global getters
  @Getter(useStore) getNameRequestLegalName!: string
  @Getter(useStore) getEditLabel!: string
  @Getter(useStore) getEditedLabel!: string
  @Getter(useStore) getEntitySnapshot!: EntitySnapshotIF
  @Getter(useStore) getEntityType!: CorpTypeCd
  @Getter(useStore) getNumberOfDirectors!: number
  @Getter(useStore) getResource!: ResourceIF
  @Getter(useStore) hasBusinessNameChanged!: boolean
  @Getter(useStore) hasBusinessTypeChanged!: boolean
  @Getter(useStore) isBcCompany!: boolean
  @Getter(useStore) isBenefitCompany!: boolean
  @Getter(useStore) isBcUlcCompany!: boolean
  @Getter(useStore) isConflictingLegalType!: boolean
  @Getter(useStore) isEntityTypeChangedByName!: boolean
  @Getter(useStore) isNameChangedByType!: boolean
  @Getter(useStore) isNumberedCompany!: boolean

  @Action(useStore) setEntityType!: (x: CorpTypeCd) => void
  @Action(useStore) setNameRequest!: (x: NameRequestIF) => void
  @Action(useStore) setNameChangedByType!: (x: boolean) => void

  selectedEntityType = null as CorpTypeCd
  confirmArticles = false
  isEditingType = false
  dropdown: boolean = null
  supportedEntityTypes: Array<string> = []

  /** Called when component is mounted. */
  mounted (): void {
    this.initializeEntityType()
    this.supportedEntityTypes = GetFeatureFlag('supported-alteration-change-business-types') || []
  }

  /** Define the entity type locally once the value has been populated in the store. */
  @Watch('getEntityType')
  private initializeEntityType (): void {
    this.selectedEntityType = this.getEntityType
  }

  /** Clear the articles confirm checkbox whenever the selected entity type changes. */
  @Watch('selectedEntityType')
  private clearConfirmArticles (): void {
    this.confirmArticles = false
  }

  /** Display the edit, so the user has to reconfirm articles. */
  @Watch('isEntityTypeChangedByName')
  entityTypeChangedByName (val): void {
    this.isEditingType = val
  }

  /** Verify New Business name. */
  get isNewName (): boolean {
    return this.getNameRequestLegalName &&
      (this.getNameRequestLegalName !== this.getEntitySnapshot?.businessInfo?.legalName)
  }

  /** Type change helper information */
  get typeChangeInfo (): string {
    return this.getResource.changeData?.typeChangeInfo
  }

  /** Entity type options based on the company type */
  get entityTypeOptions (): EntityTypeOption[] {
    const entityTypeOptions = this.getResource.changeData?.entityTypeOptions || []
    return entityTypeOptions.filter((option: EntityTypeOption) => {
      return this.supportedEntityTypes?.includes(option.value)
    })
  }

  get enableEditButton (): boolean {
    // Exclude CCC - Originally: isBcCompany || isBcUlcCompany || isBenefitCompany
    if (this.getEntitySnapshot?.businessInfo?.legalType === CorpTypeCd.BC_CCC) {
      return false
    }
    return this.supportedEntityTypes?.includes(this.getEntitySnapshot?.businessInfo?.legalType)
  }

  get nameRequestRequiredError (): boolean {
    if (this.isNumberedCompany) {
      return false
    }
    // Named companies to CC or ULC require a name request.
    if (this.isCommunityContribution || this.isUnlimitedLiability) {
      return true
    }
    // Named ULC to BC Limited require a name request.
    if (this.getEntitySnapshot?.businessInfo?.legalType === CorpTypeCd.BC_ULC_COMPANY && this.isBcLimited) {
      return true
    }
    return false
  }

  get minimumThreeDirectorError (): boolean {
    return this.isCommunityContribution && this.getNumberOfDirectors < 3
  }

  /** Reset company type values to original. */
  resetType () {
    this.setEntityType(this.getEntitySnapshot?.businessInfo?.legalType)
    // reset name request
    this.setNameRequest({
      legalType: this.getEntitySnapshot?.businessInfo?.legalType,
      legalName: this.getEntitySnapshot?.businessInfo?.legalName,
      nrNumber: this.getEntitySnapshot?.businessInfo?.nrNumber
    })
    this.setNameChangedByType(false)
    this.isEditingType = false
    this.confirmArticles = false
  }

  /** Submit new company type. */
  submitTypeChange () {
    this.setEntityType(this.selectedEntityType)
    this.isEditingType = false

    if (this.shouldUpdateName()) {
      const originalName = this.getEntitySnapshot?.businessInfo.legalName
      const updatedName = this.getUpdatedName(originalName)

      if (originalName !== updatedName) {
        const nameRequest = {
          legalType: this.selectedEntityType,
          legalName: updatedName,
          nrNumber: this.getEntitySnapshot?.businessInfo?.nrNumber
        }

        this.setNameRequest(nameRequest)
        this.setNameChangedByType(true)
      }
    }
  }

  shouldUpdateName (): boolean {
    return this.isNumberedCompany && !this.hasBusinessNameChanged
  }

  getUpdatedName (originalName: string): string {
    if (this.isBcUlcCompany) {
      originalName = originalName.replace(' LTD.', '')
      originalName += ' UNLIMITED LIABILITY COMPANY'
      return originalName
    } else if (this.isCommunityContribution) {
      originalName = originalName.replace(' LTD.', '')
      originalName += ' COMMUNITY CONTRIBUTION COMPANY'
      return originalName
    } else if (this.isBcLimited || this.isBenefitCompany) {
      originalName = originalName.replace(' UNLIMITED LIABILITY COMPANY', '').replace(' ULC', '')
      originalName = originalName.replace(' COMMUNITY CONTRIBUTION COMPANY', '').replace(' CCC', '')
      originalName = originalName.replace(' LTD.', '')
      originalName += ' LTD.'
      return originalName
    }
    return originalName
  }

  /** Check if current entity selection is a Benefit Company */
  get isBenefit (): boolean {
    return (this.selectedEntityType === CorpTypeCd.BENEFIT_COMPANY)
  }

  /** Check if current entity selection is a Unlimited Liability Company */
  get isUnlimitedLiability (): boolean {
    return (this.selectedEntityType === CorpTypeCd.BC_ULC_COMPANY)
  }

  /** Check if current entity selection is a Community Contribution Company */
  get isCommunityContribution (): boolean {
    return (this.selectedEntityType === CorpTypeCd.BC_CCC)
  }

  /** Check if current entity selection is a BC Limited Company */
  get isBcLimited (): boolean {
    return (this.selectedEntityType === CorpTypeCd.BC_COMPANY)
  }

  get updatedArticleInfo (): string {
    return ResourceUtilities.articleInfo(this.selectedEntityType)
  }

  get updatedArticleTitle (): string {
    return ResourceUtilities.articleTitle(this.selectedEntityType)
  }

  get disableDoneButton (): boolean {
    return !this.confirmArticles || this.minimumThreeDirectorError || this.nameRequestRequiredError
  }

  @Watch('isEditingType')
  @Emit('isEditingBusinessType')
  // eslint-disable-next-line @typescript-eslint/no-unused-vars
  private emitIsEditingType (isEditing: boolean): void {}
}
</script>

<style lang="scss" scoped>
@import '@/assets/styles/theme.scss';

.v-icon {
  line-height: 1.5rem
}

.actions {
  position: absolute;
  right: 0;

  .undo-action{
    border-right: 1px solid $gray1;
  }

  .v-btn {
    min-width: 0.5rem;
  }
}

.action-btns {
  margin: 15px 0;
  display: flex;
  justify-content: flex-end;

  .v-btn + .v-btn {
    margin-left: 0.5rem;
  }

  .v-btn {
    min-width: 6.5rem;
  }

  #done-btn[disabled] {
    color: white !important;
    background-color: $app-blue !important;
    opacity: 0.2;
  }
}

/*
This rule changes the color and opacity of the text in a disabled v-select component.
The :deep pseudo-element and !important flag are used to override Vuetify's default styles.
*/
:deep(.disabled-select.theme--light.v-select .v-select__selection--disabled) {
  color: $gray9 !important;
  opacity: 1 !important;
}

:deep(.theme--light.v-label) {
  font-size: .875rem;
  color: $gray7;
  font-weight: normal;
}

:deep(.v-input__slot) {
  align-items: flex-start;
}
</style>
