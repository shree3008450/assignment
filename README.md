package features.stepdef;

import com.sgcib.fdc.document.publish.document.*;
import com.sgcib.fdc.document.publish.document.secure.SecureDocumentConfig;
import com.sgcib.fdc.document.publish.document.secure.SecureDocumentDeltaFilterService;
import com.sgcib.fdc.document.publish.document.secure.SecureDocumentPublisher;
import com.sgcib.fdc.document.publish.document.secure.SecureDocumentService;
import com.sgcib.fdc.document.publish.document.secure.repository.SecureDocumentRepository;
import com.sgcib.fdc.document.publish.document.validator.DocumentValidator;
import com.sgcib.fdc.document.publish.provider.DataVaultRemoteClient;
import com.sgcib.fdc.document.publish.provider.DocumentProvider;
import com.sgcib.fdc.document.publish.provider.DocumentRetentionConfigProvider;
import com.sgcib.fdc.document.publish.remote.document.response.DocumentResponse;
import com.sgcib.fdc.document.publish.service.enricher.Enricher;
import com.sgcib.fdc.document.publish.service.handler.PathHandler;
import com.sgcib.fdc.document.publish.service.handler.PathHandlerService;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import org.junit.jupiter.api.Assertions;
import org.springframework.context.ApplicationEventPublisher;

import java.util.List;
import java.util.Optional;
import java.util.UUID;

import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.*;

public class PublishDataVaultStepDef {

    private DocumentProvider documentProvider = mock(DocumentProvider.class);
    private DocumentRetentionConfigProvider retentionConfigProvider = mock(DocumentRetentionConfigProvider.class);
    private DocumentValidator documentValidator = mock(DocumentValidator.class);
    private SecureDocumentRepository secureDocumentRepository = mock(SecureDocumentRepository.class);
    private DocumentExecutionExceptionHandler documentExecutionExceptionHandler = new DocumentExecutionExceptionHandler(secureDocumentRepository);
    private Enricher<DocumentContext> documentEnricher = mock(Enricher.class);
    private DataVaultRemoteClient dataVaultRemoteClient = mock(DataVaultRemoteClient.class);
    private ApplicationEventPublisher applicationEventPublisher = mock(ApplicationEventPublisher.class);
    private SecureDocumentPublisher secureDocumentPublisher = new SecureDocumentPublisher(applicationEventPublisher);
    private PathHandler pathHandler = mock(PathHandler.class);
    private PathHandlerService pathHandlerService = new PathHandlerService(pathHandler);
    private SecureDocumentDeltaFilterService secureDocumentDeltaFilterService = new SecureDocumentDeltaFilterService(secureDocumentRepository);
    private SecureDocumentService secureDocumentService = new SecureDocumentService(pathHandlerService, documentEnricher, secureDocumentDeltaFilterService, dataVaultRemoteClient, secureDocumentRepository, secureDocumentPublisher);
    private DocumentPublishService documentPublishService = new DocumentPublishService(documentProvider, retentionConfigProvider, documentValidator, secureDocumentService, documentExecutionExceptionHandler);
    
    private UUID documentId;
    private Document document;
    private DocumentResponse documentResponse;
    private UUID documentTypeId = UUID.randomUUID();
    private int statusCode = 0;

    @Given("I am logged into the FDC application")
    public void iAmLoggedIntoTheFdcApplication() {
        System.out.println("User logged into the FDC application.");
    }

    @Given("I have the necessary permissions to publish documents")
    public void iHaveTheNecessaryPermissionsToPublishDocuments() {
        System.out.println("User has necessary permissions to publish documents.");
    }

    @Given("I have a document with Document ID {string} ready for publication")
    public void iHaveDocumentWithDocumentIdReadyForPublication(String documentIdString) {
        this.documentId = UUID.fromString(documentIdString);
        
        // Create sample document with a document type
        documentResponse = DocumentResponse.builder()
                .id(this.documentId)
                .name("Sample Document")
                .documentType(documentTypeId)  // Set document type ID
                .businessObjects(List.of())
                .build();
                
        // Mock the document provider to return our sample document
        when(documentProvider.getDocumentById(this.documentId)).thenReturn(documentResponse);
        
        // Mock the retention config provider to return a valid config
        SecureDocumentConfig secureDocumentConfig = SecureDocumentConfig.builder()
                .documentTypeId(documentTypeId)
                .retentionDays(30)
                .checkOnExecuted(false)
                .build();
        when(retentionConfigProvider.getDocumentRetentionConfig(documentTypeId))
                .thenReturn(Optional.of(secureDocumentConfig));
                
        // Mock the document enricher to return the same context
        when(documentEnricher.enrich(any(DocumentContext.class))).thenAnswer(i -> i.getArguments()[0]);
        
        // Mock validator to do nothing (validation passes)
        doNothing().when(documentValidator).validate(any(DocumentContext.class));
    }

    @When("I select that document and click on Publish")
    public void iSelectThatDocumentAndClickOnPublish() {
        try {
            documentPublishService.publishDocument(this.documentId);
            statusCode = 204; // Successful status code
        } catch (Exception e) {
            System.err.println("Exception occurred: " + e.getMessage());
            statusCode = 500; // Error status code
        }
    }

    @Then("that document should be published successfully")
    public void thatDocumentShouldBePublishedSuccessfully() {
        verify(documentProvider, times(1)).getDocumentById(this.documentId);
        verify(retentionConfigProvider, times(1)).getDocumentRetentionConfig(documentTypeId);
        verify(documentValidator, times(1)).validate(any(DocumentContext.class));
        verify(documentEnricher, times(1)).enrich(any(DocumentContext.class));
        Assertions.assertEquals(204, statusCode);
    }

    @Then("I should receive a response with status code {int}")
    public void iShouldReceiveResponseWithStatusCode(int expectedStatusCode) {
        Assertions.assertEquals(expectedStatusCode, statusCode);
    }
}
