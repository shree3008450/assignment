isDuplicateDocument(newDocumentId: string): boolean {
  const documentArray = this.documents.controls;
  return documentArray.some(docCtrl => docCtrl.get('documentId')?.value === newDocumentId);
}
onAddDocument(): void {
  // For demonstration, assume `newDocumentId` is fetched via modal or prefilled
  const newDocumentId = prompt("Enter document ID to add");

  if (!newDocumentId) return;

  if (this.isDuplicateDocument(newDocumentId)) {
    this.toast.showToast(TOAST_STATE.WARNING, 'Duplicate Document', 'Document is already linked to same box');
    this.toast.dismissToastAfterTimeOut();
    return;
  }

  // Add new FormGroup to FormArray with the document ID
  const documentFormGroup = new FormGroup({
    documentId: new FormControl(newDocumentId),
    // ... other controls
  });

  this.documents.push(documentFormGroup);
}
