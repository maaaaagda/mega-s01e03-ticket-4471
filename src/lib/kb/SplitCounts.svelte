<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  // The desk's own view: every report counts where its tenant files it. The reports the desk read as
  // a one-day move stay in that count and are footnoted, so the rows add up to the ticket.
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const groups = $derived(groupByCause(merged, tenantOf));
  const count = (cause: string) => groups.find((g) => g.cause === cause)?.reports.length ?? 0;
  const rows = $derived([
    { problem: 'Parser reads dates month first', cause: 'parser', n: count('parser') },
    { problem: 'Booking-v3 form hints MM/DD/YYYY', cause: 'hint', n: count('hint') },
  ]);
</script>

<p><strong>{merged.length}</strong> reports in #{ticket}, <strong>{rows.length + 1}</strong> problems.</p>

<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as row (row.cause)}
      <tr><td>{row.problem}</td><td>{row.n}</td></tr>
    {/each}
  </tbody>
  <tfoot><tr><td>total</td><td>{rows.reduce((s, r) => s + r.n, 0)}</td></tr></tfoot>
</table>

<p class="note">
  A third problem sits inside these counts: {moved.length} reports read as a one-day move, counted
  under <code>{groups.find((g) => g.reports.some((r) => moved.includes(r)))?.cause}</code> where the tenant files them.
</p>
