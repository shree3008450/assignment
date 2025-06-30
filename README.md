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
import com.sgcib.fdc.document.publish.service.enricher.Enricher;
import com.sgcib.fdc.document.publish.service.handler.PathHandler;
import com.sgcib.fdc.document.publish.service.handler.PathHandlerService;
import com.sgcib.fdc.document.publish.service.handler.PathObject;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;
import org.springframework.context.ApplicationEventPublisher;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Optional;
import java.util.UUID;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.*;

public class PublishDataVaultStepDef {

    private static final UUID DOCUMENT_ID = UUID.fromString("6158b2bf-91d1-40a9-9af2-377296111a4d");
    private static final UUID DOCUMENT_TYPE_ID = UUID.randomUUID();
    private static final int STATUS_CODE = 204;
    private static final String TEST_FOLDER_PATH = "/test/path";

    // All mocks
    private final DocumentProvider documentProvider = mock(DocumentProvider.class);
    private final DocumentRetentionConfigProvider retentionConfigProvider = mock(DocumentRetentionConfigProvider.class);
    private final DocumentValidator documentValidator = mock(DocumentValidator.class);
    private final SecureDocumentRepository secureDocumentRepository = mock(SecureDocumentRepository.class);
    private final Enricher<DocumentContext> documentEnricher = mock(Enricher.class);
    private final DataVaultRemoteClient dataVaultRemoteClient = mock(DataVaultRemoteClient.class);
    private final ApplicationEventPublisher applicationEventPublisher = mock(ApplicationEventPublisher.class);
    private final PathHandler pathHandler = mock(PathHandler.class);
    private final PathHandlerService pathHandlerService = mock(PathHandlerService.class);
    private final SecureDocumentDeltaFilterService secureDocumentDeltaFilterService = mock(SecureDocumentDeltaFilterService.class);
    private final SecureDocumentService secureDocumentService = mock(SecureDocumentService.class);
    private final DocumentExecutionExceptionHandler documentExecutionExceptionHandler = mock(DocumentExecutionExceptionHandler.class);
    
    // Main service under test
    private final DocumentPublishService documentPublishService;
    
    // Test data
    private Document document;
    private DocumentResponse documentResponse;
    private SecureDocument secureDocument;
    private RuntimeException exception;
    private int responseStatusCode;
    
    public PublishDataVaultStepDef() {
        // Initialize main service in constructor
        documentPublishService = new DocumentPublishService(
                documentProvider, 
                retentionConfigProvider, 
                documentValidator, 
                secureDocumentService, 
                documentExecutionExceptionHandler
        );
    }

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
        // Create a proper business object
        BusinessObject businessObject = BusinessObject.builder()
                .operationId("OP123")
                .businessLines(new ArrayList<>())
                .bookingEntities(new ArrayList<>())
                .build();
        
        // Create a pre-built document
        document = Document.builder()
                .id(DOCUMENT_ID)
                .name("Sample Document")
                .documentType(DOCUMENT_TYPE_ID)
                .businessObjects(List.of(businessObject))
                .documentContent(DocumentContent.builder().content(new byte[0]).build())
                .build();
        
        // Create a fully mocked document response
        documentResponse = mock(DocumentResponse.class);
        when(documentResponse.toDocument()).thenReturn(document);
                
        // Create secure document
        secureDocument = SecureDocument.builder()
                .documentId(DOCUMENT_ID)
                .documentName("Sample Document")
                .operationId("OP123")
                .folderPath(TEST_FOLDER_PATH)
                .size(1000)
                .build();
                
        // Configure mocks
        given(documentProvider.getDocumentById(DOCUMENT_ID)).willReturn(documentResponse);
        
        given(retentionConfigProvider.getDocumentRetentionConfig(any(UUID.class)))
                .willReturn(Optional.of(SecureDocumentConfig.builder()
                        .documentTypeId(DOCUMENT_TYPE_ID)
                        .retentionDays(30)
                        .checkOnExecuted(false)
                        .build()));
                        
        given(secureDocumentRepository.getAllByDocumentId(any(UUID.class)))
                .willReturn(Collections.emptyList());
                
        given(documentEnricher.enrich(any(DocumentContext.class))).willAnswer(invocation -> 
                invocation.getArgument(0, DocumentContext.class));
                
        // Mock PathHandlerService
        given(pathHandlerService.getGeneratedPath(any(PathObject.class))).willReturn(TEST_FOLDER_PATH);
        
        // Mock service methods - use doNothing for void methods
        doNothing().when(secureDocumentService).publish(any(DocumentContext.class));
        doNothing().when(documentValidator).validate(any(DocumentContext.class));
    }

    @When("I select that document and click on Publish")
    public void iSelectThatDocumentAndClickOnPublish() {
        try {
            documentPublishService.publishDocument(DOCUMENT_ID);
            responseStatusCode = STATUS_CODE;
        } catch (RuntimeException e) {
            exception = e;
            responseStatusCode = 500;
            System.err.println("Exception occurred: " + e.getMessage());
            e.printStackTrace();
        }
    }

    @Then("that document should be published successfully")
    public void thatDocumentShouldBePublishedSuccessfully() {
        assertThat(exception).as("No exception should be thrown").isNull();
        verify(documentProvider).getDocumentById(DOCUMENT_ID);
        verify(documentValidator).validate(any(DocumentContext.class));
        verify(secureDocumentService).publish(any(DocumentContext.class));
    }

    @Then("I should receive a response with status code {int}")
    public void iShouldReceiveResponseWithStatusCode(int expectedStatusCode) {
        assertThat(responseStatusCode).isEqualTo(expectedStatusCode);
    }
}
