# Frontend and integration questions

1. Why Streamlit? Existing Python-native architecture.
2. What does upload handling do? Accepts user media and passes it to inference.
3. Why temporary files? Some video APIs need filesystem paths.
4. How is failure displayed? Explicit status/error messages.
5. What does the download button provide? Generated output when available.
6. What is a rerun? Streamlit re-executing after interaction.
7. Why not show fake metrics? It would misrepresent runtime evidence.
8. What happens without weights? Inference remains unavailable.
