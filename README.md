package features.stepdef;

import com.sgcib.fdc.document.publish.document.*;
import com.sgcib.fdc.document.publish.document.secure.SecureDocument;
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
import com.sgcib.fdc.document.publish.remote.document.response.RemoteBusinessObject;
import com.sgcib.fdc.document.publish.service.enricher.Enricher;
import com.sgcib.fdc.document.publish.service.handler.PathHandler;
import com.sgcib.fdc.document.publish.service.handler.PathHandlerService;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import org.junit.jupiter.api.Assertions;
import org.springframework.context.ApplicationEventPublisher;

import java.util.ArrayList;
import java.util.Collections;
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
    
    // We'll use a spy for this service to mock the generateSecureDocuments method
    private SecureDocumentDeltaFilterService secureDocumentDeltaFilterService = spy(new SecureDocumentDeltaFilterService(secureDocumentRepository));
    private SecureDocumentService secureDocumentService;
    private DocumentPublishService documentPublishService;
    
    private UUID documentId;
    private Document document;
    private DocumentResponse documentResponse;
    private UUID documentTypeId = UUID.randomUUID();
    private int statusCode = 0;
    private Exception caughtException = null;

    @Given("I am logged into the FDC application")
    public void iAmLoggedIntoTheFdcApplication() {
        System.out.println("User logged into the FDC application.");
        
        // Initialize services with spy
        secureDocumentService = spy(new SecureDocumentService(
                pathHandlerService,
                documentEnricher,
                secureDocumentDeltaFilterService,
                dataVaultRemoteClient,
                secureDocumentRepository,
                secureDocumentPublisher
        ));
        
        documentPublishService = new DocumentPublishService(
                documentProvider,
                retentionConfigProvider,
                documentValidator,
                secureDocumentService,
                documentExecutionExceptionHandler
        );
    }

    @Given("I have the necessary permissions to publish documents")
    public void iHaveTheNecessaryPermissionsToPublishDocuments() {
        System.out.println("User has necessary permissions to publish documents.");
    }

    @Given("I have a document with Document ID {string} ready for publication")
    public void iHaveDocumentWithDocumentIdReadyForPublication(String documentIdString) {
        this.documentId = UUID.fromString(documentIdString);
        
        // Create a business object with required properties
        List<RemoteBusinessObject> businessObjects = new ArrayList<>();
        RemoteBusinessObject businessObject = RemoteBusinessObject.builder()
                .operationId("OP123")
                .bookingEntities(List.of(
                    BookingEntity.builder()
                        .name("TestBookingEntity")
                        .region("EU")  // Important: region must be set to avoid RegionNotFoundException
                        .build()
                ))
                .businessLines(List.of(
                    BusinessLine.builder()
                        .name("TestBusinessLine")
                        .build()
                ))
                .build();
        businessObjects.add(businessObject);
        
        // Create sample document with business objects
        documentResponse = DocumentResponse.builder()
                .id(this.documentId)
                .name("Sample Document")
                .documentType(documentTypeId)
                .businessObjects(businessObjects)
                .build();
                
        // Mock the document provider
        when(documentProvider.getDocumentById(this.documentId)).thenReturn(documentResponse);
        
        // Mock the retention config provider
        SecureDocumentConfig secureDocumentConfig = SecureDocumentConfig.builder()
                .documentTypeId(documentTypeId)
                .retentionDays(30)
                .checkOnExecuted(false)
                .build();
        when(retentionConfigProvider.getDocumentRetentionConfig(any(UUID.class)))
                .thenReturn(Optional.of(secureDocumentConfig));
                
        // Mock the document enricher
        when(documentEnricher.enrich(any(DocumentContext.class))).thenAnswer(i -> i.getArguments()[0]);
        
        // Mock the document validator
        doNothing().when(documentValidator).validate(any(DocumentContext.class));
        
        // Mock the repository to return empty list for any document ID
        when(secureDocumentRepository.getAllByDocumentId(any(UUID.class))).thenReturn(Collections.emptyList());
        
        // Create a sample secure document that will be returned by the filter method
        SecureDocument secureDocument = SecureDocument.builder()
                .documentId(this.documentId)
                .documentName("Sample Document")
                .operationId("OP123")
                .folderPath("/test/path")
                .size(1000L)
                .build();
        
        // Mock the filter method to return our test secure document
        doReturn(List.of(secureDocument)).when(secureDocumentDeltaFilterService).filter(any());
        
        // Mock path handler service
        when(pathHandlerService.getGeneratedPath(any())).thenReturn("/test/path");
    }

    @When("I select that document and click on Publish")
    public void iSelectThatDocumentAndClickOnPublish() {
        try {
            documentPublishService.publishDocument(this.documentId);
            statusCode = 204; // Success
        } catch (Exception e) {
            caughtException = e;
            System.err.println("Exception occurred: " + e.getMessage());
            e.printStackTrace();
            statusCode = 500; // Error
        }
    }

    @Then("that document should be published successfully")
    public void thatDocumentShouldBePublishedSuccessfully() {
        Assertions.assertNull(caughtException, "No exception should be thrown");
        verify(documentProvider, times(1)).getDocumentById(this.documentId);
        verify(retentionConfigProvider, times(1)).getDocumentRetentionConfig(any(UUID.class));
        verify(documentValidator, times(1)).validate(any(DocumentContext.class));
        verify(secureDocumentService, times(1)).publish(any(DocumentContext.class));
        Assertions.assertEquals(204, statusCode);
    }

    @Then("I should receive a response with status code {int}")
    public void iShouldReceiveResponseWithStatusCode(int expectedStatusCode) {
        Assertions.assertEquals(expectedStatusCode, statusCode);
    }
}
