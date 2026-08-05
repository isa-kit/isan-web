# krtravel (Korea travel access) — schema migrations

## 2026-07-28 — Initial applet (krtravel-0728)

seedVersion `2026-07-28.1`. New applet — no prior schema, nothing to migrate
from. Tables: `districts` (25), `dong` (424), `stations` (471), `best_spots`
(25), `events` (15). Data is 100% derived from `tool/korea/build_krtravel.py`
+ `build_krtravel_assets.py` + `build_krtravel_content.py` reusing
`apps/terrain`'s already-baked `kr_stations`/`kr_district_od` content and
`network.geojson`/`seoul_districts.geojson`/`seoul_dong.geojson` geometry —
no new external fetch this round. Formulas + honesty caveats: see
`DATA_SOURCES.md` section "terrain krtravel".

Assets bundled under `apps/krtravel/assets/`: `seoul_districts.geojson` and
`seoul_dong.geojson` are copies of the terrain geometry with krtravel's
travel-ease/area-centrality/best-spots score properties joined on; `network.
geojson` is a straight copy; `national_lines.geojson`/`national_stations.
geojson` and `stations.geojson`/`events.geojson` are new point/line
FeatureCollections built from the krtravel bake outputs.
