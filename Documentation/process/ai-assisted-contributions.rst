.. SPDX-License-Identifier: GPL-2.0

===========================================
AI-Assisted Contributions to the Linux Kernel
===========================================

:Author: Sandip Poudel
:Date:   April 2026

Introduction
============

AI coding assistants — tools such as GitHub Copilot, ChatGPT, Claude,
and similar large-language-model (LLM) based systems — have become
widely used in software development.  New contributors sometimes use
them when writing their first kernel patches.  This document explains
how to use such tools responsibly, what their limitations are in the
context of kernel development, and what reviewers will expect from you
regardless of how the code was initially drafted.

This guide does **not** ban the use of AI tools.  It establishes the
standard of understanding and ownership that every patch author must
meet, whether they used AI assistance or not.


The Golden Rule: You Own the Patch
===================================

When you add your ``Signed-off-by:`` line (see
:ref:`Documentation/process/submitting-patches.rst <submittingpatches>`)
you are certifying — under the Developer Certificate of Origin (DCO) —
that you understand the patch, that you have the right to submit it,
and that it is correct to the best of your knowledge.  An AI tool
cannot sign off on your behalf.  You must be able to:

* Explain every changed line to a reviewer.
* Justify why the approach is correct in the kernel's execution context.
* Identify any introduced bugs before they reach a maintainer.

If you cannot do those three things, the patch is not ready to send.


What AI Tools Are Good At
==========================

Used carefully, AI assistants can help with:

* **Boilerplate reduction** — generating the skeleton of a new driver,
  sysfs attribute, or debugfs file that follows existing patterns.
* **Documentation drafts** — producing a first draft of kernel-doc
  comments that you then verify for accuracy.
* **Spelling and grammar** — catching typographical errors in commit
  messages and documentation.
* **Searching for similar patterns** — suggesting functions or macros
  that may already solve your problem (you must still verify they apply).


What AI Tools Get Wrong in Kernel Code
========================================

Current AI models make characteristic mistakes that are dangerous in
kernel code.  Be especially sceptical of AI-generated code in the
following areas:

Locking and concurrency
  AI models frequently produce code that acquires the wrong lock, drops
  a lock too early, or introduces ABBA deadlocks.  Always trace every
  execution path through any AI-suggested locking sequence manually.

Memory management
  Kernel memory is not garbage-collected.  AI tools commonly forget to
  call ``kfree()`` / ``put_device()`` / ``kobject_put()`` on every error
  path, or suggest ``kmalloc()`` where a slab cache or mempool is more
  appropriate.  Use sparse, smatch, or kasan to check generated code.

Error-path correctness
  ``goto``-based cleanup chains are idiomatic in the kernel.  AI tools
  often produce incomplete or misordered cleanup sequences.  Verify
  that every allocation and reference count acquired in a function is
  released on every exit path.

Kernel coding style
  The kernel's style (see :ref:`Documentation/process/coding-style.rst
  <codingstyle>`) differs from what AI models see most during training.
  Always run ``scripts/checkpatch.pl --strict`` on AI-generated code
  before sending.

Deprecated APIs
  AI models are trained on historical code and will suggest deprecated
  interfaces (e.g. ``pci_request_regions()`` patterns that have been
  superseded, or old workqueue APIs).  Cross-check any suggested API
  against the current ``include/linux/`` headers and kernel documentation.

Architecture-specific correctness
  Barrier requirements, cache-coherency assumptions, and endianness
  handling vary by architecture.  AI suggestions for these areas must
  be verified against the architecture's own documentation and existing
  drivers.


Disclosing AI Assistance
=========================

You are not required to disclose AI tool use in a patch.  However, if
a substantial portion of the code was drafted by an AI assistant, it is
good practice — and increasingly expected by some subsystem maintainers
— to mention it in the cover letter of a patch series, for example::

    This series was initially drafted with assistance from [tool name].
    All logic has been reviewed and verified by the author.

Never claim code written by an AI tool as your own creative work in a
copyright notice.  The patch's copyright belongs to you insofar as you
made the creative selections, restructuring, and verification — not to
the AI tool.


Pre-Submission Checklist for AI-Assisted Patches
=================================================

Before sending any patch that involved AI assistance, confirm each item:

.. list-table::
   :widths: 10 90
   :header-rows: 0

   * - ☐
     - I can explain every changed line without referring to the AI tool.
   * - ☐
     - I ran ``scripts/checkpatch.pl --strict`` and resolved all warnings.
   * - ☐
     - I built the kernel with ``CONFIG_WERROR=y`` and there are no new
       compiler warnings.
   * - ☐
     - I ran the code through at least one static analyser (sparse,
       smatch, or coccinelle) and addressed the findings.
   * - ☐
     - I traced every error path and confirmed that resources are freed.
   * - ☐
     - I reviewed the relevant subsystem mailing list for prior art and
       confirmed no equivalent patch is already queued.
   * - ☐
     - My ``Signed-off-by:`` reflects genuine understanding of the patch.


Further Reading
===============

* :ref:`Documentation/process/submitting-patches.rst <submittingpatches>`
* :ref:`Documentation/process/coding-style.rst <codingstyle>`
* :ref:`Documentation/process/kernel-enforcement-statement.rst <kernelenforcement>`
* ``scripts/checkpatch.pl`` — in-tree style checker
* ``Documentation/dev-tools/`` — static analysis and testing tools
