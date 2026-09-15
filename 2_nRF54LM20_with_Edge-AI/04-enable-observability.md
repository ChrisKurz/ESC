# (Optional) Exercise model observability

To enable you should enable `CONFIG_MODELS_OBSERVABILITY` in the prj.conf (Step 5)

Additional library-level options to check:
- `CONFIG_MEMFAULT_NCS_PROJECT_KEY` — your Memfault project key (required if observability + Memfault upload is used).
- `CONFIG_NRF_EDGEAI_OBSV_MAX_CLASSES` — number of classes the observed model predicts; must match your model.
- `CONFIG_NRF_EDGEAI_OBSV_MEMFAULT_AUTO_COLLECT_INTERVAL_SEC` — automatic collection interval (shorten this for testing, together with enabling server debug mode for the device in the Memfault dashboard).

If you plan to enable observability, also review Section 4's metric options (`CONFIG_NRF_EDGEAI_OBSV_METRIC_PROBS_DISTRIBUTION`, `CONFIG_NRF_EDGEAI_OBSV_METRIC_TRANSITION_MATRIX`, and related bin-count settings) so the on-device buffers and the CBOR encode buffer are sized correctly.

1. Connect a Memfault gateway to the device — either the nRF Connect Device Manager app or the Memfault Web Bluetooth Client — following the same procedure as for the Peripheral Memfault Diagnostic Service (MDS) sample.
2. Wait for the interval set by `CONFIG_NRF_EDGEAI_OBSV_MEMFAULT_AUTO_COLLECT_INTERVAL_SEC` (shorten it for faster testing).
3. In the Memfault web dashboard, open **CDR Payloads** and filter by device name/time to find a payload with reason `edgeai_observability`.
4. Download the payload.
5. Decode it on a host PC with the provided helper script, in binary mode:
   ```
   ./scripts/decode_edgeai_obsv_cdr/decode_edgeai_obsv_cdr.py --binary --file <payload>.bin
   ```
6. Inspect the decoded probability distribution and (for the KWS stage) transition matrix to evaluate real-world model behavior — confidence levels per class, and which keywords are being confused with each other.
