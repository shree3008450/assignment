 get documents(): FormArray {
  const docs = this.boxForm.get('documents') as FormArray;

  if (docs.length > 0) {
    const lastDocGroup = docs.at(docs.length - 1);
    const docId = lastDocGroup.get('documentId')?.value;

    if (this.box?.secureDocumentIds?.length && docId) {
      const isDuplicate = this.box.secureDocumentIds.includes(docId);

      if (isDuplicate) {
        this.toast.showToast(
          TOAST_STATE.WARNING,
          'Validation Error',
          'Document is already linked to same box'
        );
        this.toast.dismissToastAfterTimeOut();
      }
    }
  }

  return docs;
}
