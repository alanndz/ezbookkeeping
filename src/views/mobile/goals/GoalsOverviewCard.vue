<template>
    <f7-card class="goals-ov-card" :class="{ 'skeleton-text': loading && !hasAnyData }">
        <f7-card-header>
            <span>{{ tt('Goals Overview') }}</span>
            <f7-link href="/goals" class="goals-ov-view-all">{{ tt('View All') }}</f7-link>
        </f7-card-header>

        <f7-card-content :padding="false">
            <div v-if="loading && !hasAnyData" class="goals-ov-skeleton">
                <div v-for="i in 3" :key="i" class="goals-ov-skeleton-row skeleton-effect-wave"></div>
            </div>

            <div v-else-if="!loading && !hasAnyData" class="goals-ov-empty">
                <span class="text-color-gray">{{ tt('No goals yet') }}</span>
            </div>

            <div v-else v-for="goal in goals" :key="goal.id" class="goals-ov-row">
                <div class="goals-ov-header-row">
                    <span class="goals-ov-name">{{ goal.name }}</span>
                    <span class="goals-ov-balance" :class="balanceColorClass(goal)">{{ fmtBalance(goal) }}</span>
                </div>
                <div class="goals-ov-progress-wrap">
                    <div class="goals-ov-progress-track">
                        <div
                            class="goals-ov-progress-fill"
                            :class="progressColorClass(goal)"
                            :style="{ width: progressPct(goal) + '%' }"
                        />
                    </div>
                    <span class="goals-ov-progress-pct">{{ progressPct(goal) }}%</span>
                </div>
                <div class="goals-ov-target-row">
                    <span class="goals-ov-target">{{ tt('Target:') }} {{ fmtAmount(goal.targetAmount) }}</span>
                </div>
            </div>
        </f7-card-content>
    </f7-card>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useI18n } from '@/locales/helpers.ts';
import { useSettingsStore } from '@/stores/setting.ts';
import { useUserStore } from '@/stores/user.ts';
import { useAccountsStore } from '@/stores/account.ts';
import { DISPLAY_HIDDEN_AMOUNT } from '@/consts/numeral.ts';

export interface GoalOverviewItem {
    id: string;
    name: string;
    accountId: string;
    targetAmount: number;
    targetDate: number;
    createdAt: number;
}

const props = defineProps<{
    loading: boolean;
    goals: GoalOverviewItem[];
}>();

const { tt, formatAmountToLocalizedNumeralsWithCurrency } = useI18n();
const settingsStore = useSettingsStore();
const userStore = useUserStore();
const accountsStore = useAccountsStore();

const showAmountInHomePage = computed<boolean>(() => settingsStore.appSettings.showAmountInHomePage);
const defaultCurrency = computed<string>(() => userStore.currentUserDefaultCurrency);
const hasAnyData = computed<boolean>(() => props.goals && props.goals.length > 0);

function accountBalance(accountId: string): number {
    return accountsStore.allPlainAccounts.find(a => a.id === accountId)?.balance ?? 0;
}

function accountCurrency(accountId: string): string {
    return accountsStore.allPlainAccounts.find(a => a.id === accountId)?.currency ?? defaultCurrency.value;
}

function progressPct(goal: GoalOverviewItem): number {
    if (goal.targetAmount <= 0) return 100;
    const bal = accountBalance(goal.accountId);
    return Math.min(100, Math.round((bal / goal.targetAmount) * 100));
}

function progressColorClass(goal: GoalOverviewItem): string {
    const pct = progressPct(goal);
    if (pct >= 100) return 'goals-ov-progress-green';
    if (pct >= 80) return 'goals-ov-progress-blue';
    if (pct >= 50) return 'goals-ov-progress-amber';
    return 'goals-ov-progress-red';
}

function balanceColorClass(goal: GoalOverviewItem): string {
    const pct = progressPct(goal);
    if (pct >= 100) return 'text-color-green';
    if (pct >= 50) return 'text-color-orange';
    return 'text-color-red';
}

function fmtBalance(goal: GoalOverviewItem): string {
    if (!showAmountInHomePage.value) {
        return formatAmountToLocalizedNumeralsWithCurrency(DISPLAY_HIDDEN_AMOUNT, accountCurrency(goal.accountId));
    }
    const bal = accountBalance(goal.accountId);
    const cur = accountCurrency(goal.accountId);
    return formatAmountToLocalizedNumeralsWithCurrency(bal, cur).replace(/[,.]00$/, '');
}

function fmtAmount(cents: number): string {
    if (!showAmountInHomePage.value) {
        return formatAmountToLocalizedNumeralsWithCurrency(DISPLAY_HIDDEN_AMOUNT, defaultCurrency.value);
    }
    return formatAmountToLocalizedNumeralsWithCurrency(cents, defaultCurrency.value).replace(/[,.]00$/, '');
}
</script>

<style>
.goals-ov-card .card-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.goals-ov-view-all {
    font-size: 13px;
}

.goals-ov-skeleton {
    padding: 8px 16px 12px;
}

.goals-ov-skeleton-row {
    height: 52px;
    border-radius: 16px;
    background: var(--f7-list-border-color);
    margin-bottom: 10px;
}

.goals-ov-empty {
    padding: 20px 16px;
    text-align: center;
    font-size: 14px;
}

.goals-ov-row {
    padding: 10px 16px;
    border-bottom: 1px solid var(--f7-list-border-color);
}

.goals-ov-row:last-child {
    border-bottom: none;
}

.goals-ov-header-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 6px;
}

.goals-ov-name {
    font-size: 14px;
    font-weight: 500;
    flex: 1;
    min-width: 0;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.goals-ov-balance {
    font-size: 13px;
    font-weight: 600;
    flex-shrink: 0;
    font-variant-numeric: tabular-nums;
}

.goals-ov-progress-wrap {
    display: flex;
    align-items: center;
    gap: 8px;
    margin-bottom: 4px;
}

.goals-ov-progress-track {
    flex: 1;
    height: 6px;
    border-radius: 3px;
    background-color: rgba(0, 0, 0, 0.12);
    overflow: hidden;
}

.goals-ov-progress-fill {
    height: 100%;
    border-radius: 3px;
    transition: width 0.3s ease;
}

.goals-ov-progress-red    { background-color: var(--f7-color-red); }
.goals-ov-progress-amber  { background-color: var(--f7-color-orange); }
.goals-ov-progress-blue   { background-color: var(--f7-theme-color); }
.goals-ov-progress-green  { background-color: var(--f7-color-green); }

.goals-ov-progress-pct {
    font-size: 0.75rem;
    width: 36px;
    text-align: right;
    flex-shrink: 0;
    opacity: 0.6;
}

.goals-ov-target-row {
    display: flex;
    justify-content: space-between;
}

.goals-ov-target {
    font-size: 0.75rem;
    opacity: 0.55;
}

.goals-ov-card .card-content {
    padding-bottom: 4px;
}
</style>
