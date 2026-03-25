components_v2_bridge
=====================

Components V2 bridge.

Converts legacy ``embed=`` / ``embeds=`` kwargs into a :class:`discord.ui.LayoutView`
containing :class:`discord.ui.Container` children so existing cog code never
needs to know about Components V2.

Each embed becomes one Container whose children are built in this order:

1. **Thumbnail** (``embed.thumbnail``) — a ``discord.ui.Section`` whose
   ``accessory`` is a ``discord.ui.Thumbnail`` (Discord requires thumbnails on
   sections), with title / description / fields / footer as the section body.
   ``embed.footer`` text is shown; footer ``icon_url`` is intentionally omitted
   (V2 ``TextDisplay`` does not mirror classic footer icons cleanly).
2. **Text** — without a thumbnail, one ``TextDisplay`` for the same markdown.
3. **Fields** — non-inline fields are stacked blocks; up to three consecutive
   **inline** fields share one row (name line + value line, `` · ``-separated).
4. **Main image** (``embed.image``) — a single-item ``discord.ui.MediaGallery``
   appended after the text block.
5. **Buttons / selects** from any accompanying ``view=`` — migrated out of the
   v1 :class:`discord.ui.View` into ``discord.ui.ActionRow`` items and appended
   inside the same Container so they are visually grouped with the embed
   content.

When a cog passes ``view=`` with a classic :class:`discord.ui.View`, the bridge
consumes it and folds its components into the Container. The original
``view=`` is replaced by the new ``LayoutView``.

Usage
-----

Call once at bot startup, before any cogs are loaded::

    from components_v2_bridge import enable_components_v2_embed_bridge
    enable_components_v2_embed_bridge()

Requirements
------------

``discord.py >= 2.5`` (or a fork that ships ``discord.ui.LayoutView`` /
``Container`` / ``TextDisplay`` / ``MediaGallery`` / ``Thumbnail`` /
``ActionRow``). Older builds that lack these classes fall back gracefully to
classic embeds — no crash, no data loss.
