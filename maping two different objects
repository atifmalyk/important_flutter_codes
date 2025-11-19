requestBecomeInvestor.contractDocuments = contractsController.simaDocuments.value
    .map((s((sima) => ContractDocuments(
          documentName: sima.documentName,
          documentContent: sima.documentPdf,        // this is the PDF content
          documentStatus: sima.status,              // "U", "S", etc.
          documentId: null,                         // backend will generate or not needed for upload
          clientId: null,                           // or pass actual clientId if you have it
        ))
    .toList();
